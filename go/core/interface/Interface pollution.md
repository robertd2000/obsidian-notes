https://100go.co/5-interface-pollution/

Interfaces are one of the cornerstones of the Go language when designing and structuring our code. However, like many tools or concepts, abusing them is generally not a good idea. Interface pollution is about overwhelming our code with unnecessary abstractions, making it harder to understand. It’s a common mistake made by developers coming from another language with different habits. Before delving into the topic, let’s refresh our minds about Go’s interfaces. Then, we will see when it’s appropriate to use interfaces and when it may be considered pollution.

## Concepts

An interface provides a way to specify the behavior of an object. We use interfaces to create common abstractions that multiple objects can implement. What makes Go interfaces so different is that they are satisfied implicitly. There is no explicit keyword like `implements` to mark that an object X implements interface Y.

To understand what makes interfaces so powerful, we will dig into two popular ones from the standard library: `io.Reader` and `io.Writer`. The `io` package provides abstractions for I/O primitives. Among these abstractions, `io.Reader` relates to reading data from a data source and `io.Writer` to writing data to a target, as represented in the next figure:

[![](https://100go.co/img/ioreaderwriter.svg)](https://100go.co/img/ioreaderwriter.svg)

The `io.Reader` contains a single Read method:

`[](https://100go.co/5-interface-pollution/#__codelineno-0-1)type Reader interface {     [](https://100go.co/5-interface-pollution/#__codelineno-0-2)    Read(p []byte) (n int, err error) [](https://100go.co/5-interface-pollution/#__codelineno-0-3)}`

Custom implementations of the `io.Reader` interface should accept a slice of bytes, filling it with its data and returning either the number of bytes read or an error.

On the other hand, `io.Writer` defines a single method, Write:

`[](https://100go.co/5-interface-pollution/#__codelineno-1-1)type Writer interface {     [](https://100go.co/5-interface-pollution/#__codelineno-1-2)    Write(p []byte) (n int, err error) [](https://100go.co/5-interface-pollution/#__codelineno-1-3)}`

Custom implementations of `io.Writer` should write the data coming from a slice to a target and return either the number of bytes written or an error. Therefore, both interfaces provide fundamental abstractions:

- `io.Reader` reads data from a source
- `io.Writer` writes data to a target

What is the rationale for having these two interfaces in the language? What is the point of creating these abstractions?

Let’s assume we need to implement a function that should copy the content of one file to another. We could create a specific function that would take as input two `*os.File`. Or, we can choose to create a more generic function using `io.Reader` and `io.Writer` abstractions:

`[](https://100go.co/5-interface-pollution/#__codelineno-2-1)func copySourceToDest(source io.Reader, dest io.Writer) error {     [](https://100go.co/5-interface-pollution/#__codelineno-2-2)    // ... [](https://100go.co/5-interface-pollution/#__codelineno-2-3)}`

This function would work with `*os.File` parameters (as `*os.File` implements both `io.Reader` and `io.Writer`) and any other type that would implement these interfaces. For example, we could create our own `io.Writer` that writes to a database, and the code would remain the same. It increases the genericity of the function; hence, its reusability.

Furthermore, writing a unit test for this function is easier because, instead of having to handle files, we can use the `strings` and `bytes` packages that provide helpful implementations:

`[](https://100go.co/5-interface-pollution/#__codelineno-3-1)func TestCopySourceToDest(t *testing.T) {     [](https://100go.co/5-interface-pollution/#__codelineno-3-2)    const input = "foo"    [](https://100go.co/5-interface-pollution/#__codelineno-3-3)    source := strings.NewReader(input) // Creates an io.Reader    [](https://100go.co/5-interface-pollution/#__codelineno-3-4)    dest := bytes.NewBuffer(make([]byte, 0)) // Creates an io.Writer [](https://100go.co/5-interface-pollution/#__codelineno-3-5)    [](https://100go.co/5-interface-pollution/#__codelineno-3-6)    err := copySourceToDest(source, dest) // Calls copySourceToDest from a *strings.Reader and a *bytes.Buffer    [](https://100go.co/5-interface-pollution/#__codelineno-3-7)    if err != nil {        [](https://100go.co/5-interface-pollution/#__codelineno-3-8)        t.FailNow()    [](https://100go.co/5-interface-pollution/#__codelineno-3-9)    } [](https://100go.co/5-interface-pollution/#__codelineno-3-10)    [](https://100go.co/5-interface-pollution/#__codelineno-3-11)    got := dest.String()    [](https://100go.co/5-interface-pollution/#__codelineno-3-12)    if got != input {        [](https://100go.co/5-interface-pollution/#__codelineno-3-13)        t.Errorf("expected: %s, got: %s", input, got)    [](https://100go.co/5-interface-pollution/#__codelineno-3-14)    } [](https://100go.co/5-interface-pollution/#__codelineno-3-15)}`

In the example, source is a `*strings.Reader`, whereas dest is a `*bytes.Buffer`. Here, we test the behavior of `copySourceToDest` without creating any files.

While designing interfaces, the granularity (how many methods the interface contains) is also something to keep in mind. A [known proverb](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=318s) in Go relates to how big an interface should be:

Rob Pike

The bigger the interface, the weaker the abstraction.

Indeed, adding methods to an interface can decrease its level of reusability. `io.Reader` and `io.Writer` are powerful abstractions because they cannot get any simpler. Furthermore, we can also combine fine-grained interfaces to create higher-level abstractions. This is the case with `io.ReadWriter`, which combines the reader and writer behaviors:

`[](https://100go.co/5-interface-pollution/#__codelineno-4-1)type ReadWriter interface {     [](https://100go.co/5-interface-pollution/#__codelineno-4-2)    Reader    [](https://100go.co/5-interface-pollution/#__codelineno-4-3)    Writer [](https://100go.co/5-interface-pollution/#__codelineno-4-4)}`

Note

As Einstein said, “_Everything should be made as simple as possible, but no simpler._” Applied to interfaces, this denotes that finding the perfect granularity for an interface isn’t necessarily a straightforward process.

Let’s now discuss common cases where interfaces are recommended.

## When to use interfaces

When should we create interfaces in Go? Let’s look at three concrete use cases where interfaces are usually considered to bring value. Note that the goal isn’t to be exhaustive because the more cases we add, the more they would depend on the context. However, these three cases should give us a general idea:

- Common behavior
- Decoupling
- Restricting behavior

### Common behavior

The first option we will discuss is to use interfaces when multiple types implement a common behavior. In such a case, we can factor out the behavior inside an interface. If we look at the standard library, we can find many examples of such a use case. For example, sorting a collection can be factored out via three methods:

- Retrieving the number of elements in the collection
- Reporting whether one element must be sorted before another
- Swapping two elements

Hence, the following interface was added to the `sort` package:

`[](https://100go.co/5-interface-pollution/#__codelineno-5-1)type Interface interface {     [](https://100go.co/5-interface-pollution/#__codelineno-5-2)    Len() int // Number of elements    [](https://100go.co/5-interface-pollution/#__codelineno-5-3)    Less(i, j int) bool // Checks two elements    [](https://100go.co/5-interface-pollution/#__codelineno-5-4)    Swap(i, j int) // Swaps two elements [](https://100go.co/5-interface-pollution/#__codelineno-5-5)}`

This interface has a strong potential for reusability because it encompasses the common behavior to sort any collection that is index-based.

Throughout the `sort` package, we can find dozens of implementations. If at some point we compute a collection of integers, for example, and we want to sort it, are we necessarily interested in the implementation type? Is it important whether the sorting algorithm is a merge sort or a quicksort? In many cases, we don’t care. Hence, the sorting behavior can be abstracted, and we can depend on the `sort.Interface`.

Finding the right abstraction to factor out a behavior can also bring many benefits. For example, the `sort` package provides utility functions that also rely on `sort.Interface`, such as checking whether a collection is already sorted. For instance:

`[](https://100go.co/5-interface-pollution/#__codelineno-6-1)func IsSorted(data Interface) bool {     [](https://100go.co/5-interface-pollution/#__codelineno-6-2)    n := data.Len()    [](https://100go.co/5-interface-pollution/#__codelineno-6-3)    for i := n - 1; i > 0; i-- {        [](https://100go.co/5-interface-pollution/#__codelineno-6-4)        if data.Less(i, i-1) {            [](https://100go.co/5-interface-pollution/#__codelineno-6-5)            return false        [](https://100go.co/5-interface-pollution/#__codelineno-6-6)        }    [](https://100go.co/5-interface-pollution/#__codelineno-6-7)    }    [](https://100go.co/5-interface-pollution/#__codelineno-6-8)    return true [](https://100go.co/5-interface-pollution/#__codelineno-6-9)}`

Because `sort.Interface` is the right level of abstraction, it makes it highly valuable.

Let’s now see another main use case when using interfaces.

### Decoupling

Another important use case is about decoupling our code from an implementation. If we rely on an abstraction instead of a concrete implementation, the implementation itself can be replaced with another without even having to change our code. This is the Liskov Substitution Principle (the L in Robert C. Martin’s [SOLID](https://en.wikipedia.org/wiki/SOLID) design principles).

One benefit of decoupling can be related to unit testing. Let’s assume we want to implement a `CreateNewCustomer` method that creates a new customer and stores it. We decide to rely on the concrete implementation directly (let’s say a `mysql.Store` struct):

`[](https://100go.co/5-interface-pollution/#__codelineno-7-1)type CustomerService struct {     [](https://100go.co/5-interface-pollution/#__codelineno-7-2)    store mysql.Store // Depends on the concrete implementation [](https://100go.co/5-interface-pollution/#__codelineno-7-3)} [](https://100go.co/5-interface-pollution/#__codelineno-7-4)[](https://100go.co/5-interface-pollution/#__codelineno-7-5)func (cs CustomerService) CreateNewCustomer(id string) error {     [](https://100go.co/5-interface-pollution/#__codelineno-7-6)    customer := Customer{id: id}    [](https://100go.co/5-interface-pollution/#__codelineno-7-7)    return cs.store.StoreCustomer(customer) [](https://100go.co/5-interface-pollution/#__codelineno-7-8)}`

Now, what if we want to test this method? Because `customerService` relies on the actual implementation to store a `Customer`, we are obliged to test it through integration tests, which requires spinning up a MySQL instance (unless we use an alternative technique such as [`go-sqlmock`](https://github.com/DATA-DOG/go-sqlmock), but this isn’t the scope of this section). Although integration tests are helpful, that’s not always what we want to do. To give us more flexibility, we should decouple `CustomerService` from the actual implementation, which can be done via an interface like so:

`[](https://100go.co/5-interface-pollution/#__codelineno-8-1)type customerStorer interface { // Creates a storage abstraction     [](https://100go.co/5-interface-pollution/#__codelineno-8-2)    StoreCustomer(Customer) error [](https://100go.co/5-interface-pollution/#__codelineno-8-3)} [](https://100go.co/5-interface-pollution/#__codelineno-8-4)[](https://100go.co/5-interface-pollution/#__codelineno-8-5)type CustomerService struct {     [](https://100go.co/5-interface-pollution/#__codelineno-8-6)    storer customerStorer // Decouples CustomerService from the actual implementation [](https://100go.co/5-interface-pollution/#__codelineno-8-7)} [](https://100go.co/5-interface-pollution/#__codelineno-8-8)[](https://100go.co/5-interface-pollution/#__codelineno-8-9)func (cs CustomerService) CreateNewCustomer(id string) error {     [](https://100go.co/5-interface-pollution/#__codelineno-8-10)    customer := Customer{id: id}    [](https://100go.co/5-interface-pollution/#__codelineno-8-11)    return cs.storer.StoreCustomer(customer) [](https://100go.co/5-interface-pollution/#__codelineno-8-12)}`

Because storing a customer is now done via an interface, this gives us more flexibility in how we want to test the method. For instance, we can:

- Use the concrete implementation via integration tests
- Use a mock (or any kind of test double) via unit tests
- Or both

Let’s now discuss another use case: to restrict a behavior.

### Restricting behavior

The last use case we will discuss can be pretty counterintuitive at first sight. It’s about restricting a type to a specific behavior. Let’s imagine we implement a custom configuration package to deal with dynamic configuration. We create a specific container for `int` configurations via an `IntConfig` struct that also exposes two methods: `Get` and `Set`. Here’s how that code would look:

`[](https://100go.co/5-interface-pollution/#__codelineno-9-1)type IntConfig struct {     [](https://100go.co/5-interface-pollution/#__codelineno-9-2)    // ... [](https://100go.co/5-interface-pollution/#__codelineno-9-3)} [](https://100go.co/5-interface-pollution/#__codelineno-9-4)[](https://100go.co/5-interface-pollution/#__codelineno-9-5)func (c *IntConfig) Get() int {     [](https://100go.co/5-interface-pollution/#__codelineno-9-6)    // Retrieve configuration [](https://100go.co/5-interface-pollution/#__codelineno-9-7)} [](https://100go.co/5-interface-pollution/#__codelineno-9-8)[](https://100go.co/5-interface-pollution/#__codelineno-9-9)func (c *IntConfig) Set(value int) {     [](https://100go.co/5-interface-pollution/#__codelineno-9-10)    // Update configuration [](https://100go.co/5-interface-pollution/#__codelineno-9-11)}`

Now, suppose we receive an `IntConfig` that holds some specific configuration, such as a threshold. Yet, in our code, we are only interested in retrieving the configuration value, and we want to prevent updating it. How can we enforce that, semantically, this configuration is read-only, if we don’t want to change our configuration package? By creating an abstraction that restricts the behavior to retrieving only a config value:

`[](https://100go.co/5-interface-pollution/#__codelineno-10-1)type intConfigGetter interface {     [](https://100go.co/5-interface-pollution/#__codelineno-10-2)    Get() int [](https://100go.co/5-interface-pollution/#__codelineno-10-3)}`

Then, in our code, we can rely on `intConfigGetter` instead of the concrete implementation:

`[](https://100go.co/5-interface-pollution/#__codelineno-11-1)type Foo struct {     [](https://100go.co/5-interface-pollution/#__codelineno-11-2)    threshold intConfigGetter [](https://100go.co/5-interface-pollution/#__codelineno-11-3)} [](https://100go.co/5-interface-pollution/#__codelineno-11-4)[](https://100go.co/5-interface-pollution/#__codelineno-11-5)func NewFoo(threshold intConfigGetter) Foo { // Injects the configuration getter     [](https://100go.co/5-interface-pollution/#__codelineno-11-6)    return Foo{threshold: threshold} [](https://100go.co/5-interface-pollution/#__codelineno-11-7)} [](https://100go.co/5-interface-pollution/#__codelineno-11-8)[](https://100go.co/5-interface-pollution/#__codelineno-11-9)func (f Foo) Bar()  {     [](https://100go.co/5-interface-pollution/#__codelineno-11-10)    threshold := f.threshold.Get() // Reads the configuration    [](https://100go.co/5-interface-pollution/#__codelineno-11-11)    // ... [](https://100go.co/5-interface-pollution/#__codelineno-11-12)}`

In this example, the configuration getter is injected into the `NewFoo` factory method. It doesn’t impact a client of this function because it can still pass an `IntConfig` struct as it implements `intConfigGetter`. Then, we can only read the configuration in the `Bar` method, not modify it. Therefore, we can also use interfaces to restrict a type to a specific behavior for various reasons, such as semantics enforcement.

In this section, we saw three potential use cases where interfaces are generally considered as bringing value: factoring out a common behavior, creating some decoupling, and restricting a type to a certain behavior. Again, this list isn’t exhaustive, but it should give us a general understanding of when interfaces are helpful in Go.

Now, let’s finish this section and discuss the problems with interface pollution.

## Interface pollution

It’s fairly common to see interfaces being overused in Go projects. Perhaps the developer’s background was C# or Java, and they found it natural to create interfaces before concrete types. However, this isn’t how things should work in Go.

As we discussed, interfaces are made to create abstractions. And the main caveat when programming meets abstractions is remembering that **abstractions should be discovered, not created**. What does this mean? It means we shouldn’t start creating abstractions in our code if there is no immediate reason to do so. We shouldn’t design with interfaces but wait for a concrete need. Said differently, we should create an interface when we need it, not when we foresee that we could need it.

What’s the main problem if we overuse interfaces? The answer is that they make the code flow more complex. Adding a useless level of indirection doesn’t bring any value; it creates a worthless abstraction making the code more difficult to read, understand, and reason about. If we don’t have a strong reason for adding an interface and it’s unclear how an interface makes a code better, we should challenge this interface’s purpose. Why not call the implementation directly?

Note

We may also experience performance overhead when calling a method through an interface. It requires a lookup in a hash table’s data structure to find the concrete type an interface points to. But this isn’t an issue in many contexts as the overhead is minimal.

In summary, we should be cautious when creating abstractions in our code—abstractions should be discovered, not created. It’s common for us, software developers, to overengineer our code by trying to guess what the perfect level of abstraction is, based on what we think we might need later. This process should be avoided because, in most cases, it pollutes our code with unnecessary abstractions, making it more complex to read.

Rob Pike

Don’t design with interfaces, discover them.

Let’s not try to solve a problem abstractly but solve what has to be solved now. Last, but not least, if it’s unclear how an interface makes the code better, we should probably consider removing it to make our code simpler.