https://webdevstation.com/posts/understanding-golang-context/

When working with concurrent operations in Go, few topics are as important—and as misunderstood—as the **`context`** package. Whether you are building an HTTP API, orchestrating background workers, or integrating with external services, _golang context_ is the idiomatic way to propagate cancellation signals, enforce timeouts, carry deadlines, and pass request-scoped values.

In this article we will demystify the `context` package, walk through common use-cases, and share production-tested best practices.

## Why Context Exists

Go’s lightweight goroutines make it trivial to spin up concurrent work, but once you have hundreds (or thousands) of goroutines you need a structured way to:

1. Cancel unfinished work when a client disconnects or a parent task ends.
2. Enforce upper time bounds to prevent runaway operations.
3. Propagate deadlines deep into the call graph.
4. Attach request-level metadata (trace IDs, auth tokens, etc.) without polluting function signatures.

The `context` package solves these problems with two core ideas:

- **Cancellation propagation** via `Done()` channels.
- **Immutable trees**—each derived context references its parent, forming a hierarchy that can be cancelled from the root.

## Creating and Cancelling Contexts

The building blocks are `context.Background()` (or `context.TODO()`), `context.WithCancel`, `context.WithTimeout`, and `context.WithDeadline`.

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // Start with a root context
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel() // always release resources

    // Fire off a worker goroutine
    go func() {
        select {
        case <-ctx.Done():
            fmt.Println("worker cancelled:", ctx.Err())
            return
        }
    }()

    // Simulate some condition that requires cancellation
    time.Sleep(500 * time.Millisecond)
    cancel()

    time.Sleep(100 * time.Millisecond)
}
```

Running this prints:

```
worker cancelled: context canceled
```

### Timeout Helper

`context.WithTimeout` wraps `WithCancel` plus a timer:

```go
ctx, cancel := context.WithTimeout(parent, 2*time.Second)
// After 2s: ctx.Err() == context.DeadlineExceeded
```

Remember to **always call the returned `cancel`**—even when the timeout expires—so the timer’s internal resources are freed.

## Passing Context Down the Call Stack

The first parameter of every context-aware function should be `ctx context.Context`:

```go
func fetch(ctx context.Context, url string) ([]byte, error) {
    req, _ := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    return io.ReadAll(resp.Body)
}
```

When `ctx` is cancelled upstream, `http.Client` aborts the request automatically.

## Deadlines vs. Timeouts

A **deadline** is an absolute moment (`2025-06-16T14:05:00+02:00`) while a **timeout** is a relative duration (`5s`). Internally both are implemented with `WithDeadline`, but modelling them correctly communicates intent:

```go
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(parent, deadline)
```

## Storing Values in Context

`context.WithValue` allows passing request-scoped data without modifying every function signature. Use it sparingly:

```go
// Key type prevents collisions
 type key string
 const traceIDKey key = "traceID"

 ctx := context.WithValue(parent, traceIDKey, "abc-123")

 // Downstream retrieval
 if v := ctx.Value(traceIDKey); v != nil {
     traceID := v.(string)
     log.Println("traceID:", traceID)
 }
```

### Guidelines

1. Only store immutable, request-specific data (IDs, auth tokens).
2. Never store optional params that belong in function arguments.
3. Define unexported key types to avoid collisions across packages.

## Common Pitfalls

|Pitfall|Solution|
|---|---|
|Returning `nil` context|Accept a `context.Context` argument and demand callers pass one.|
|Forgetting to cancel|Always `defer cancel()` after `WithCancel / WithTimeout / WithDeadline`.|
|Blocking select without `<-ctx.Done()`|Include cancellation in every `select` that may block.|
|Misusing `WithValue` for configs|Pass explicit parameters instead.|

## End-to-End Example: HTTP Server with Timeouts

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "time"
)

func main() {
    srv := &http.Server{
        Addr:         ":8080",
        ReadTimeout:  3 * time.Second,
        WriteTimeout: 5 * time.Second,
        Handler:      http.HandlerFunc(handler),
    }

    // Shutdown gracefully on interrupt
    go func() {
        <-time.After(10 * time.Second) // Simulate interrupt
        ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
        defer cancel()
        _ = srv.Shutdown(ctx)
    }()

    fmt.Println("Serving on :8080")
    if err := srv.ListenAndServe(); err != http.ErrServerClosed {
        panic(err)
    }
    fmt.Println("Server gracefully stopped")
}

func handler(w http.ResponseWriter, r *http.Request) {
    // r.Context() inherits deadlines from the server
    select {
    case <-time.After(2 * time.Second):
        w.Write([]byte("done"))
    case <-r.Context().Done():
        http.Error(w, "request cancelled", http.StatusRequestTimeout)
    }
}
```

## Best Practices Checklist

- Pass `context.Context` as the first parameter; never embed it in structs.
- Do not store contexts—pass them along the call chain.
- Cancel contexts to free resources early.
- Use short-lived timeouts close to I/O boundaries rather than a single large timeout at the root.
- Keep functions context-aware; return early on `<-ctx.Done()`.

## Conclusion

The _golang context_ package brings order to concurrent Go programs by standardising how we propagate cancellation and deadlines. Mastering it unlocks more reliable, resource-efficient applications.