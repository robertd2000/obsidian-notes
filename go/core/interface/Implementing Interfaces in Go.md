https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/

This comprehensive guide introduces the concept of interfaces in Go, explaining what interfaces are, why they are used, and how to implement them. It covers basic and advanced concepts, practical examples, and real-world applications, making it ideal for beginners and intermediate Go programmers.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#introduction-to-interfaces)Introduction to Interfaces

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#what-is-an-interface)What is an Interface?

In the world of programming, interfaces are a way to define a set of related methods that can be implemented by any data type or struct. Think of interfaces like a blueprint for actions that can be performed. Just like how a blueprint outlines the structure of a building, an interface outlines the structure of what a type can do. In Go, interfaces are a fundamental and powerful feature that enable you to write more flexible and reusable code.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#why-use-interfaces)Why Use Interfaces?

Interfaces are incredibly useful for achieving polymorphism in Go, a statically typed language. Polymorphism allows you to write more generic code that can handle different types without needing to know the exact type at compile time. This flexibility is invaluable, especially in large applications where components need to work with various data types interchangeably. By using interfaces, you can write code that is more modular, extendable, and easier to maintain.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#basics-of-interface-implementation)Basics of Interface Implementation

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#defining-an-interface)Defining an Interface

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#interface-declaration-syntax)Interface Declaration Syntax

In Go, an interface is defined using the `interface` keyword, followed by a set of method signatures enclosed in curly braces. Here’s a simple example of an interface declaration:

```go
type Shape interface {
    Area() float64
    Perimeter() float64
}
```

In this example, `Shape` is an interface that declares two methods: `Area()` and `Perimeter()`. Any type that implements these two methods is said to satisfy the `Shape` interface.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#multiple-methods-in-an-interface)Multiple Methods in an Interface

Interfaces can have any number of methods. Let’s create an interface for a printer:

```go
type Printer interface {
    Print()
    Scan()
    Fax()
}
```

This `Printer` interface requires any implementing type to have `Print()`, `Scan()`, and `Fax()` methods.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#implementing-an-interface)Implementing an Interface

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#implicit-and-explicit-implementations)Implicit and Explicit Implementations

One of the key features of interfaces in Go is that implementation is implicit. A type implements an interface if it has all the methods that the interface requires. There’s no need to explicitly declare that a type implements an interface, making the code cleaner and more intuitive.

Let’s see an implicit implementation in action:

```go
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}
```

In this example, the `Rectangle` struct has methods `Area()` and `Perimeter()`, which match the methods required by the `Shape` interface. Therefore, `Rectangle` implicitly satisfies the `Shape` interface.

Explicit implementation is rare in Go, mainly used for clarity. Here’s how you might do it:

```go
var _ Shape = (*Rectangle)(nil)
```

This line of code explicitly states that a pointer to `Rectangle` satisfies the `Shape` interface. It’s not necessary for the code to work but can be helpful for documentation and error checking at compile time.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#methods-with-receivers)Methods with Receivers

Interfaces are essentially about defining a certain behavior. When a type has methods with the same signatures as those in an interface, it implicitly implements that interface. Methods with receivers are crucial here:

```go
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * 3.14159 * c.Radius
}
```

Here, the `Circle` struct also satisfies the `Shape` interface because it has `Area()` and `Perimeter()` methods with the appropriate signatures.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#example-basic-interface-implementation)Example: Basic Interface Implementation

Let’s put everything together with a basic example:

```go
package main

import (
    "fmt"
)

type Shape interface {
    Area() float64
    Perimeter() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * 3.14159 * c.Radius
}

func main() {
    shapes := []Shape{
        Rectangle{Width: 10, Height: 5},
        Circle{Radius: 5},
    }

    for _, shape := range shapes {
        fmt.Printf("Area: %f, Perimeter: %f\n", shape.Area(), shape.Perimeter())
    }
}
```

In this example, both `Rectangle` and `Circle` satisfy the `Shape` interface. This allows us to create a slice of `Shape` and iterate over it, calling the `Area()` and `Perimeter()` methods on each shape. The output will be:

```
Area: 50.000000, Perimeter: 30.000000
Area: 78.539750, Perimeter: 31.415900
```

This example demonstrates the flexibility of interfaces, allowing different concrete types (like `Rectangle` and `Circle`) to be treated uniformly.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#advanced-interface-concepts)Advanced Interface Concepts

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#interface-satisfactions)Interface Satisfactions

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#how-interfaces-work-under-the-hood)How Interfaces Work Under the Hood

Interfaces in Go are interfaces in the traditional object-oriented sense, but they are implemented in a more lightweight and type-safe manner. At runtime, an interface value stores a concrete value and the methods associated with that value. This means that you can pass around interface values and call the methods on them without knowing what the underlying concrete type is.

When a type implements an interface, Go’s runtime system checks if the type has all the required methods. If it does, the type satisfies the interface. This check is done at compile time, ensuring type safety.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#empty-interfaces)Empty Interfaces

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#usage-in-generic-programming)Usage in Generic Programming

An empty interface is an interface that has no methods. It is represented by `interface{}`. Any type can be assigned to an empty interface because all types satisfy an empty interface by definition.

Empty interfaces are often used in generic programming to write functions that can accept arguments of any type:

```go
func PrintType(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Type: int, Value: %d\n", v)
    case float64:
        fmt.Printf("Type: float64, Value: %f\n", v)
    case string:
        fmt.Printf("Type: string, Value: %s\n", v)
    default:
        fmt.Printf("Unknown type\n")
    }
}

func main() {
    PrintType(42)
    PrintType(3.14159)
    PrintType("Hello, World!")
}
```

In this example, the `PrintType()` function takes an `interface{}` and prints the type and value of the argument. The `switch` statement uses type assertions to determine the actual type of the argument.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#type-assertions-and-type-switches)Type Assertions and Type Switches

Type assertions are used to extract a concrete value from an interface value. They take the form `i.(Type)` where `i` is an interface value and `Type` is the desired concrete type.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#type-assertions)Type Assertions

Consider the following example:

```go
func main() {
    var i interface{} = "hello"

    s := i.(string)
    fmt.Println(s)

    f, ok := i.(float64)
    if !ok {
        fmt.Println("Type assertion failed for float64")
    } else {
        fmt.Println(f)
    }
}
```

The first type assertion `i.(string)` does not have the `ok` syntax and asserts that `i` is a string. If it is not, the program will panic. The second type assertion `i.(float64)` uses the `ok` syntax, which returns a boolean indicating whether the assertion was successful. Since `i` is a string, not a float64, `ok` will be `false` and the program will print "Type assertion failed for float64".

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#type-switches)Type Switches

Type switches allow you to switch on the type of an interface value. They are useful when you have an interface value and you don’t know its underlying type.

Here’s an example of a type switch:

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Type: int, Value: %d\n", v)
    case float64:
        fmt.Printf("Type: float64, Value: %f\n", v)
    case string:
        fmt.Printf("Type: string, Value: %s\n", v)
    default:
        fmt.Println("Unknown type")
    }
}

func main() {
    describe(42)
    describe(3.14159)
    describe("Hello, World!")
    describe(true)
}
```

This `describe()` function uses a type switch to handle different types of interface values. The output will be:

```
Type: int, Value: 42
Type: float64, Value: 3.141590
Type: string, Value: Hello, World!
Unknown type
```

Type switches are a powerful tool for handling different types within the same block of code, especially when the type is not known until runtime.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#working-with-interfaces-in-functions)Working with Interfaces in Functions

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#interface-as-a-parameter)Interface as a Parameter

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#benefits-of-using-interfaces)Benefits of Using Interfaces

Using interfaces as function parameters is one of the most common and powerful uses of interfaces in Go. By using interfaces, functions can operate on any type that implements the given interface, making the code more flexible and reusable.

For example, consider a function that prints the area of a shape:

```go
package main

import (
    "fmt"
)

type Shape interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func PrintArea(s Shape) {
    fmt.Printf("Area of shape: %f\n", s.Area())
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    circle := Circle{Radius: 5}

    PrintArea(rect)   // Output: Area of shape: 50.000000
    PrintArea(circle) // Output: Area of shape: 78.539750
}
```

In this example, `PrintArea()` takes a `Shape` interface as a parameter. It can accept any type that implements the `Shape` interface, such as `Rectangle` or `Circle`. This flexibility is a key advantage of using interfaces.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#example-interface-in-a-function)Example: Interface in a Function

Let’s expand on the previous example by adding a `Perimeter` method and using it in a function:

```go
package main

import (
    "fmt"
)

type Shape interface {
    Area() float64
    Perimeter() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * 3.14159 * c.Radius
}

func DescribeShape(s Shape) {
    fmt.Printf("Area: %f, Perimeter: %f\n", s.Area(), s.Perimeter())
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    circle := Circle{Radius: 5}

    DescribeShape(rect)
    DescribeShape(circle)
}
```

In this extended example, `DescribeShape()` takes a `Shape` interface and prints both the area and perimeter of the shape. This function can be used with any type that satisfies the `Shape` interface, making it highly reusable.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#returning-interfaces-from-functions)Returning Interfaces from Functions

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#when-to-return-an-interface)When to Return an Interface

Returning interfaces from functions is useful when you want to abstract away the concrete type and only expose the behaviors defined by the interface. This is particularly useful in large applications where the concrete type might change over time or is not known at compile time.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#example-interface-as-a-return-type)Example: Interface as a Return Type

Here’s an example of a function that returns an interface:

```go
package main

import (
    "math/rand"
    "fmt"
)

type Shape interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func RandomShape() Shape {
    if rand.Intn(2) == 0 {
        return Rectangle{Width: 10, Height: 5}
    } else {
        return Circle{Radius: 5}
    }
}

func main() {
    shape := RandomShape()
    fmt.Printf("Area of random shape: %f\n", shape.Area())
}
```

In this example, `RandomShape()` returns a `Shape` interface. It returns either a `Rectangle` or a `Circle` at random. The calling code doesn’t need to know the exact type; it only needs to know that the type implements the `Shape` interface and has an `Area()` method. This flexibility is a hallmark of Go’s interface system.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#interface-embedding)Interface Embedding

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#embedding-interfaces)Embedding Interfaces

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#how-to-embed-interfaces)How to Embed Interfaces

Interface embedding in Go allows you to include one interface’s methods within another. By embedding, you can compose interfaces in a flexible and powerful way.

Here’s an example of interface embedding:

```go
package main

import "fmt"

type Shape interface {
    Area() float64
    Perimeter() float64
}

type ColoredShape interface {
    Shape
    Color() string
}

type ColoredRectangle struct {
    Rectangle
    ColorName string
}

func (cr ColoredRectangle) Color() string {
    return cr.ColorName
}

func main() {
    rect := ColoredRectangle{Rectangle{Width: 10, Height: 5}, "Blue"}
    fmt.Printf("Area: %f, Perimeter: %f, Color: %s\n", rect.Area(), rect.Perimeter(), rect.Color())
}
```

In this example, `ColoredShape` embeds the `Shape` interface and adds a `Color()` method. The `ColoredRectangle` struct embeds a `Rectangle` and implements the `Color()` method, satisfying the `ColoredShape` interface.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#embedding-multiple-interfaces)Embedding Multiple Interfaces

You can also embed multiple interfaces in a single interface:

```go
type Printer interface {
    Print()
}

type Scanner interface {
    Scan()
}

type MultifunctionDevice interface {
    Printer
    Scanner
    Copy()
}

type OfficeDevice struct {
    Model string
}

func (o OfficeDevice) Print() {
    fmt.Println("Printing...")
}

func (o OfficeDevice) Scan() {
    fmt.Println("Scanning...")
}

func (o OfficeDevice) Copy() {
    fmt.Println("Copying...")
}

func main() {
    device := OfficeDevice{Model: "X123"}
    var mdev MultifunctionDevice = device
    mdev.Print()    // Output: Printing...
    mdev.Scan()     // Output: Scanning...
    mdev.Copy()     // Output: Copying...
}
```

In this example, `MultifunctionDevice` embeds `Printer` and `Scanner` interfaces and adds a `Copy()` method. The `OfficeDevice` struct implements all three methods, satisfying the `MultifunctionDevice` interface.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#practical-uses-of-interface-embedding)Practical Uses of Interface Embedding

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#example-using-embedded-interfaces)Example: Using Embedded Interfaces

Interface embedding allows you to define complex interfaces by combining simpler ones. This makes your codebase more modular and easier to manage.

Consider a complex system with different types of devices. You can define base interfaces like `Printable`, `Scannable`, and `Faxable` and combine them using embedding:

```go
package main

import "fmt"

type Printable interface {
    Print()
}

type Scannable interface {
    Scan()
}

type Faxable interface {
    Fax()
}

type MultifunctionDevice interface {
    Printable
    Scannable
    Faxable
}

type OfficeDevice struct {
    Model string
}

func (o OfficeDevice) Print() {
    fmt.Println("Printing...")
}

func (o OfficeDevice) Scan() {
    fmt.Println("Scanning...")
}

func (o OfficeDevice) Fax() {
    fmt.Println("Faxing...")
}

func main() {
    device := OfficeDevice{Model: "X123"}
    var mdev MultifunctionDevice = device
    mdev.Print() // Output: Printing...
    mdev.Scan()  // Output: Scanning...
    mdev.Fax()   // Output: Faxing...
}
```

In this example, `MultifunctionDevice` embeds `Printable`, `Scannable`, and `Faxable` interfaces. The `OfficeDevice` struct implements all three methods, satisfying the `MultifunctionDevice` interface.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#advanced-interface-concepts-1)Advanced Interface Concepts

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#interface-satisfactions-1)Interface Satisfactions

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#how-interfaces-work-under-the-hood-1)How Interfaces Work Under the Hood

When a type satisfies an interface, Go’s runtime system checks if the type has all the methods declared in the interface. This check is done at compile time, ensuring type safety. Interfaces are implemented implicitly in Go, meaning a type satisfies an interface simply by implementing the required methods.

In Go, an interface value is a pair of a value and a concrete type. The value is a pointer to the underlying data, and the concrete type is the type of the underlying data. This mechanism allows interfaces to work efficiently without the performance overhead of dynamic typing.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#empty-interfaces-1)Empty Interfaces

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#usage-in-generic-programming-1)Usage in Generic Programming

Empty interfaces are one of the most powerful features of Go. An empty interface can hold a value of any type because every type in Go automatically implements at least zero methods. Here’s an example:

```go
package main

import "fmt"

func PrintValue(i interface{}) {
    fmt.Printf("Type: %T, Value: %v\n", i, i)
}

func main() {
    PrintValue(42)
    PrintValue(3.14159)
    PrintValue("Hello, World!")
    PrintValue(true)
}
```

The `PrintValue()` function takes an `interface{}` as a parameter and prints the type and value of the argument. This function can accept any type, demonstrating the power of empty interfaces for generic programming.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#working-with-interfaces-in-functions-1)Working with Interfaces in Functions

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#interface-as-a-parameter-1)Interface as a Parameter

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#benefits-of-using-interfaces-1)Benefits of Using Interfaces

Using interfaces as function parameters allows you to write more flexible and reusable code. This approach is particularly useful when designing APIs or libraries because it allows the users of the API to provide their own types as long as they satisfy the required interface.

Consider an API that performs operations on shapes. By using a `Shape` interface, the API can handle any shape type, as long as it has the required methods. This flexibility makes the code more extensible and easier to maintain.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#example-interface-in-a-function-1)Example: Interface in a Function

Here’s a practical example of using interfaces in a function:

```go
package main

import (
    "fmt"
)

type Shape interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func PrintArea(s Shape) {
    fmt.Printf("Area: %f\n", s.Area())
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    circle := Circle{Radius: 5}

    PrintArea(rect)   // Output: Area: 50.000000
    PrintArea(circle) // Output: Area: 78.539750
}
```

In this example, `PrintArea()` takes a `Shape` interface and prints the area of the shape. The function can work with any type that implements the `Shape` interface, making it highly flexible.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#returning-interfaces-from-functions-1)Returning Interfaces from Functions

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#when-to-return-an-interface-1)When to Return an Interface

Returning interfaces from functions is useful when you want to abstract away the concrete type and only expose the behavior defined by the interface. This is helpful in scenarios where the exact type is determined at runtime or when you want to hide the implementation details.

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#example-interface-as-a-return-type-1)Example: Interface as a Return Type

Here’s an example of returning an interface from a function:

```go
package main

import (
    "fmt"
    "math/rand"
)

type Shape interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func RandomShape() Shape {
    if rand.Intn(2) == 0 {
        return Rectangle{Width: 10, Height: 5}
    } else {
        return Circle{Radius: 5}
    }
}

func main() {
    shape := RandomShape()
    fmt.Printf("Random shape area: %f\n", shape.Area())
}
```

In this example, `RandomShape()` returns a `Shape` interface. It returns either a `Rectangle` or a `Circle` at random. The calling code doesn’t need to know the exact type; it only needs to work with the `Shape` interface.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#real-world-applications)Real-world Applications

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#use-cases-for-interfaces)Use Cases for Interfaces

Interfaces are used extensively in real-world Go applications for variety of reasons:

- **Decoupling Code:** Interfaces allow you to decouple different parts of your application, making it easier to maintain and evolve over time.
- **Testing:** Interfaces make it easy to write mock implementations for testing purposes.
- **Extensibility:** Interfaces allow you to add new types and behaviors without modifying existing code.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#real-world-examples)Real-world Examples

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#example-strategy-pattern-in-go)Example: Strategy Pattern in Go

The strategy pattern is a design pattern that enables selecting an algorithm’s behavior at runtime. It allows a method to be swapped out at runtime by using a variable of the strategy interface type.

Here’s a simple example of the strategy pattern using interfaces:

```go
package main

import (
    "fmt"
)

type PaymentStrategy interface {
    Pay(amount float64)
}

type CreditCard struct {
    CardNumber string
}

func (c CreditCard) Pay(amount float64) {
    fmt.Printf("Paid $%f using credit card %s\n", amount, c.CardNumber)
}

type PayPal struct {
    Email string
}

func (p PayPal) Pay(amount float64) {
    fmt.Printf("Paid $%f using PayPal account %s\n", amount, p.Email)
}

type Payment struct {
    Strategy PaymentStrategy
}

func (p *Payment) SetStrategy(strategy PaymentStrategy) {
    p.Strategy = strategy
}

func (p *Payment) ExecutePayment(amount float64) {
    p.Strategy.Pay(amount)
}

func main() {
    payment := Payment{}
    payment.SetStrategy(CreditCard{CardNumber: "1234-5678-9012-3456"})
    payment.ExecutePayment(100.00)

    payment.SetStrategy(PayPal{Email: "user@example.com"})
    payment.ExecutePayment(50.00)
}
```

In this example, `PaymentStrategy` is an interface with a `Pay()` method. `CreditCard` and `PayPal` are structs that implement the `Pay()` method. The `Payment` struct has a `PaymentStrategy` and methods to set the strategy and execute payment. This setup allows the payment method to be changed at runtime, demonstrating the strategy pattern using interfaces.

## [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#conclusion)Conclusion

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#summary-of-key-points)Summary of Key Points

- **Interfaces** in Go are used to define a set of methods that a type can implement.
- Interfaces in Go are implemented implicitly; a type satisfies an interface by implementing its methods.
- Interfaces are particularly useful for achieving polymorphism and writing flexible, reusable code.
- **Type assertions** and **type switches** provide ways to work with specific types when the interface type is not sufficient.
- **Interface embedding** allows you to compose complex interfaces by combining simpler ones, making your code more modular and easier to maintain.
- Interfaces are extensively used in real-world applications for decoupling, testing, and extensibility.

### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#further-reading)Further Reading

#### [](https://golang.ntxm.org/docs/structs-and-interfaces/implementing-interfaces/#recommended-reading-materials)Recommended Reading Materials

- **Effective Go:** A great starting point for understanding best practices and idiomatic Go programming. It covers interfaces in detail and provides practical examples.
- **Go Lang Specification:** The official specification provides a thorough explanation of interfaces and other Go language features.
- **GoByExample:** An excellent resource with practical examples and explanations of various Go concepts, including interfaces.

By mastering interfaces in Go, you can write more flexible, maintainable, and scalable code. Interfaces are a powerful feature that can significantly improve the design and architecture of your applications. Happy coding!