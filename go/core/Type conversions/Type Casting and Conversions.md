https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di

Go, also known as Golang, is a statically typed language. This means that the type of every variable is known at compile time, providing safety and predictability in your code. However, this also requires that any conversion from one type to another is explicit and deliberate. In this article, we’ll explore the various type casting and conversion mechanisms available in Go, from basic numeric conversions to more complex interface and pointer conversions.

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#1-basic-type-conversions)1. Basic Type Conversions

Go allows conversion between basic types like integers, floating-point numbers, and strings, but these conversions must be done explicitly.

### [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#numeric-types)Numeric Types

Conversions between different numeric types are straightforward but must be explicit:  

```go
var i int = 42
var f float64 = float64(i)  // int to float64
var u uint = uint(i)        // int to uint
```

In this example, we convert an int to a `float64` and to a `uint`. These conversions are explicit because Go does not perform automatic (implicit) type conversions.

### [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#string-and-byte-slice)String and Byte Slice

Go strings are immutable, but they can be converted to and from byte slices (`[]byte`), which are mutable:  

```go
var s string = "hello"
var b []byte = []byte(s)   // string to []byte
var s2 string = string(b)  // []byte to string
```

Similarly, you can convert between strings and rune slices (`[]rune`), where rune is a type alias for `int32`:  

```go
var r []rune = []rune(s)   // string to []rune
var s3 string = string(r)  // []rune to string
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#2-custom-type-conversions)2. Custom Type Conversions

In Go, you can define your own types based on existing ones. Conversions between custom types and their underlying types are explicit:  

```go
type MyInt int
var i int = 10
var mi MyInt = MyInt(i)   // int to MyInt
var i2 int = int(mi)      // MyInt to int
```

This explicit conversion is necessary to ensure that the compiler can verify the safety of your code.

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#3-pointer-conversions)3. Pointer Conversions

Pointers in Go reference the memory address of a variable. You can convert between a value and its pointer:  

```go
var x int = 42
var p *int = &x     // int to *int (pointer to int)
var y int = *p      // *int to int (dereferencing)
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#4-interface-type-conversions)4. Interface Type Conversions

Interfaces in Go are used to define a set of methods. You can convert between concrete types and interfaces:  

```go
var a interface{} = 42    // int to interface{}
var b int = a.(int)       // interface{} to int (type assertion)
```

### [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#type-assertions)Type Assertions

A type assertion provides access to an interface's concrete value:  

```go
if v, ok := a.(int); ok {
    fmt.Println("a is an int:", v)
}
```

### [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#type-switch)Type Switch

A type switch allows you to perform different actions based on the dynamic type of an interface:  

```go
switch v := a.(type) {
case int:
    fmt.Println("a is an int:", v)
case string:
    fmt.Println("a is a string:", v)
default:
    fmt.Println("a is of unknown type")
}
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#5-unsafe-conversions)5. Unsafe Conversions

The unsafe package allows you to bypass Go’s type safety, enabling conversions that would otherwise be illegal:  

```go
import "unsafe"

var i int = 42
var p *int = &i
var fp *float64 = (*float64)(unsafe.Pointer(p))  // *int to *float64
```

> Warning: Unsafe conversions should be used sparingly and only when absolutely necessary, as they can lead to undefined behavior.

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#6-channel-type-conversions)6. Channel Type Conversions

Channels are a powerful feature in Go, allowing communication between goroutines. You can convert between bidirectional and unidirectional channels:  

```go
ch := make(chan int)
var sendOnlyChan chan<- int = ch  // bidirectional to send-only
var recvOnlyChan <-chan int = ch  // bidirectional to receive-only
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#7-struct-and-array-conversions)7. Struct and Array Conversions

Conversions between structs or arrays with identical layouts require explicit casting:  

```go
type Point struct {
    X, Y int
}

type Coord struct {
    X, Y int
}

var p Point = Point{1, 2}
var c Coord = Coord(p)  // Convert Point to Coord (same field types)
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#8-slice-conversions)8. Slice Conversions

Slices are references to arrays, and while you can convert between slices of the same type, converting between different types of slices requires an explicit conversion:  

```go
var a []int = []int{1, 2, 3}
var b []int = a[1:]  // Convert a slice to another slice of the same type
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#9-nil-interface-conversions)9. Nil Interface Conversions

A `nil` value in Go can be assigned to any `interface` type:  

```go
var x interface{} = nil
var y error = nil
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#10-function-type-conversions)10. Function Type Conversions

Go functions can be converted to different types, provided the signatures are compatible:  

```go
type FuncType func(int) int

func square(x int) int {
    return x * x
}

var f FuncType = FuncType(square)  // Convert function to FuncType
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#11-arraytoslice-conversion)11. Array-to-Slice Conversion

You can create a slice from an array, which is essentially a reference to the array:  

```go
var arr [5]int = [5]int{1, 2, 3, 4, 5}
var sl []int = arr[:]  // Convert array to slice
```

## [](https://dev.to/zakariachahboun/a-comprehensive-guide-to-type-casting-and-conversions-in-go-26di#conclusion)Conclusion

Type casting and conversions in Go are explicit by design, making the code safer and easier to understand. By requiring explicit conversions, Go helps prevent subtle bugs that can arise from implicit type coercion, common in some other programming languages. Understanding these conversions and using them correctly is crucial for writing robust and efficient Go programs.