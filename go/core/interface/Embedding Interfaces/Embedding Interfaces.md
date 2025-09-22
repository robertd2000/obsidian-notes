Create new interfaces by combining existing ones, promoting composition and reusability. Embedded interface methods automatically included. Enables interface hierarchies from simpler, focused interfaces. Supports composition over inheritance for modular, extensible systems.

## 1. Interfaces in Go:

### [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#11-what-are-interfaces)1.1. What are Interfaces?

In Go, an interface is a type that specifies a set of method signatures. When a concrete type provides definitions for all the methods in an interface, it is said to implement the interface.

### [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#12-defining-interfaces)1.2. Defining Interfaces:

You define an interface using the `type` keyword followed by the interface name and the `interface` keyword:  

```go
type Writer interface {
    Write([]byte) (int, error)
}
```

### [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#13-significance-in-gos-type-system)1.3. Significance in Go's Type System:

Interfaces in Go allow you to:

- Define behavior that types should fulfill.
- Enable polymorphic behavior. You can write functions and methods that accept interface types, and then pass values of any concrete type that satisfies the interface.
- Provide a way to define contracts. If a type implements an interface, it guarantees certain methods with defined signatures are present in the type.

### [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#14-interface-values)1.4. Interface Values:

Interface values can hold any value that implements the specified methods. An interface value has two components: a value and a concrete type. When we call a method on an interface value, the method of its underlying type is executed.  

```go
var w Writer
w = os.Stdout
w.Write([]byte("Hello, Go!\n"))
```

In the above example, `os.Stdout` implements the `Writer` interface, so we can assign it to the `w` variable. When we call `w.Write`, it calls `os.Stdout.Write`.

## [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#2-embedding-in-go)2. Embedding in Go:

### [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#21-what-is-embedding)2.1. What is Embedding?

Embedding allows one struct type to include another struct, inheriting the fields and methods of the embedded type. It is Go's mechanism to achieve composition over traditional inheritance.

### [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#22-how-to-embed)2.2. How to Embed:

To embed a type, you declare a field in the struct without a field name, just the type:  

```go
type Address struct {
    Street, City, State string
}

type Person struct {
    Name string
    Address
}

p := Person{
    Name:    "Alice",
    Address: Address{"123 Main St", "Anytown", "CA"},
}
fmt.Println(p.Street)  // Output: 123 Main St
```

In the above code, `Person` embeds `Address`. This means that a `Person` not only has a `Name` but also has `Street`, `City`, and `State` due to the embedded `Address`.

### [](https://dev.to/diwakarkashyap/interfaces-and-embedding-in-golang-go-2em4#23-inheritancelike-behavior)2.3. Inheritance-like Behavior:

Embedding provides a way to "inherit" methods. If the embedded type has methods, the embedding type will have those methods too, provided it doesn't define its own methods with the same name.  

```go
func (a Address) FullAddress() string {
    return a.Street + ", " + a.City + ", " + a.State
}

// Even though Person doesn't define FullAddress, it gains the method through embedding Address.
address := p.FullAddress()
```

However, Go doesn't support classical inheritance where you can extend and override base class methods. Instead, Go promotes composition over inheritance, making systems easier to understand and maintain.

To summarize:

- Interfaces in Go allow types to adhere to contracts and enable polymorphism.
- Embedding allows structs to inherit fields and methods from other structs, giving a mechanism for composition over classical inheritance.

