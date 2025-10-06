https://dev.to/mcaci/how-to-use-the-context-done-method-in-go-22me

Here is an example of a basic usage of the `context` package to signal the completion of a task executed in a goroutine:  

```go
package main

import (
    "context"
    "log"
    "time"
)

const interval = 500

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go func() {
        time.Sleep(5 * interval * time.Millisecond)
        cancel()
    }()
    f(ctx)
}

func f(ctx context.Context) {
    ticker := time.NewTicker(interval * time.Millisecond)
    for {
        select {
        case <-ticker.C:
            doSomething()
        case <-ctx.Done():
            return
        }
    }
}

func doSomething() { log.Println("tick") }
```

[Playground link](https://play.golang.org/p/kzCPf9OD50k)

In this example `doSomething()` is executed 5 times before the `time.Sleep(...)` function completes and the `cancel` function created by the `context.WithCancel(...)` sends a value to the channel `ctx.Done()` that will end the `for` loop and exit.

`context.Background()` is used as a base for creating a new context variable as described in the context package [documentation](https://golang.org/pkg/context/#Background).

This is a useful way to signal the completion of a task executed in a goroutine and in this context is a powerful alternative to the usage of `sync.waitGroup`.