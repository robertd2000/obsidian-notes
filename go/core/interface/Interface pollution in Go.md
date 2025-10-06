https://rakyll.org/interface-pollution/

If there was a single powerful language feature in Go, it’d be the interfaces. The internals of Go contain strong combinations of useful ideas from various type systems and inevitably they ring the curiosity bells. I recently surveyed Github for Go interface declarations, and the results indicated that Go users pollute the environment with interfaces no one needs or will use.

Don’t export any interfaces until you have to.

Interfaces are great, but interface pollution is not so. You’re likely to come to Go from a language (if not from a dynamic language) that generates a static dispatch table during compilation, the compiler require you to explicitly tell the interfaces a type wants to implement. That’s how the complier can generate a vtable with pointers to all available virtual functions. If your background is in C++ or Java, you’re likely to have some baggage around initiating your codebase with abstract types and work on the concrete implementation as a follow-up exercise. This is not how you do it in Go. Introduce concrete types and don’t export any interfaces unless you have to encourage external packages to implement one. io package is a good starting point to study some of the the best practices. It exports interfaces because it also needs to export generic-use functions like Copy.

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```

Should your package export generic functionality? If the answer is a “maybe”, you’re likely to be polluting your package with an interface declaration. Justify the need of multiple implementations, likeliness of them to interact back with your package and act accordingly.

Go doesn’t have a traditional dispatch table, and can rely on the interface values during a method dispatch. It’s literally more of a freestyle dispatcher mechanism that requires some work during interface value assignment — it generates a tiny lookup hash-table for the concrete type it’s pointing to. The assignment is not insanely expensive, so it’s a fair exchange for a more pleasant type system. Ian Lance Taylor has [a great blog post about the internals](http://www.airs.com/blog/archives/277) if you’re looking for further reading.

If a user requires some level of “inversion of control”, an on-the-fly interface definition in their own scope would just work. This possibility minimizes the presumptions you have to make about the way your package is being consumed and the initial abstractions you have to work on.

It also applies to the testability concerns, you don’t have to provide interfaces to help the user to write their own stubs. Earlier today, I got a request to export an interface from the pubsub package to make it more mockable. Rather than doing so, the preferable way is to tell your user to introduce an interface that focuses on the calls they want to write a stub for. Point to the actual implementation via an interface value.

```go
type acknowledger interface {
    Ack(sub string, id ...string) error
}

type mockClient struct{}

func (c *mockClient) Ack(sub string, id ...string) error {
    return nil
}

var acker acknowledger = pubsub.New(...)

acker = &mockClient{} // in the test package
```

Noteworthy that, in Go, the standard library defines tiny interfaces you happen to implement without effort and is doing a good job at encouraging the developers to write compatible code with the rest of the standard library — and the other third party packages. Adopt what’s available in the standard library where possible and document accordingly.

Go fascinates me each time I deeply reevaluate my experience with the interfaces. Given the chance of minimal fragmentation in interfaces, this is how a programming language make software, that isn’t designed to work together, work well together.