https://blog.logrocket.com/understanding-generics-go-1-18/

The addition of generics is the most significant change to Go (formerly Golang) since its debut. The Go community has long been asking for generics as a feature since the inception of the language, and now it’s finally here.

![Understanding Generics In Go 1.18](https://blog.logrocket.com/wp-content/uploads/2022/03/understanding-generics-go-1.18.png)

Go generics’ implementation is very different from traditional implementations found in C++, while bearing similarities to Rust’s generics implementation — we’ll be taking a look at understanding generics in Go in this overview article.

## What are generics?

To be able to use generics properly, one needs to understand what generics are and why they are required. Generics allow you to write code without explicitly providing specific data types they take or returns — in other words, while writing some code or data structure, you don’t provide the type of values.

These type values are instead passed later. Generics allow Go programmers to specify types later and avoid the boilerplate code.

## Why generics?

The aim of generics is to reduce boilerplate code. For example, a reverse array function doesn’t require knowing the type of element of the array, but without generics, there is no type-safe method of representing this without repetition. You instead have to implement a reverse function for each type, which will create a huge amount of code that needs to be in sync with each type implementation maintained accordingly.

This problem is what is ultimately solved by generics.

- [Generics syntax](https://blog.logrocket.com/understanding-generics-go-1-18/#generic-syntax)
- [Type parameters](https://blog.logrocket.com/understanding-generics-go-1-18/#type-parameters)
- [Type constraints](https://blog.logrocket.com/understanding-generics-go-1-18/#type-constraints)
- [Type approximation](https://blog.logrocket.com/understanding-generics-go-1-18/#type-approximation)
- [`constraints` package](https://blog.logrocket.com/understanding-generics-go-1-18/#constraints-package)
- [Interfaces vs. generics](https://blog.logrocket.com/understanding-generics-go-1-18/#interfaces-vs-generics)

## Generics syntax

Go `1.18.0` introduces a new syntax for providing additional metadata about types and defining constraints on these types.

```go
package main

import "fmt"

func main() {
        fmt.Println(reverse([]int{1, 2, 3, 4, 5}))
}

// T is a type parameter that is used like normal type inside the function
// any is a constraint on type i.e T has to implement "any" interface
func reverse[T any](s []T) []T {
        l := len(s)
        r := make([]T, l)

        for i, ele := range s {
                r[l-i-1] = ele
        }
        return r
}
```


[Playground link](https://go.dev/play/p/mqrxpG0xTfx)

![Type Parameter Type Constraint Visualized](https://blog.logrocket.com/wp-content/uploads/2022/03/type-parameter-type-constraint-visualized.png)

As you can see in the above image,`[]` brackets are used to specify type parameters, which are a list of identifiers and a constraint interface. Here, `T` is a type parameter that is used to define arguments and return the type of the function.

The parameter is also accessible inside the function. `any` is an interface; `T` has to implement this interface. Go 1.18 introduces `any` as an alias to `interface{}`.

The type parameter is like a type variable — all the operations supported by normal types are supported by type variables (for example, `make` function). The variable initialized using these type parameters will support the operation of the constraint; in the above example, the constraint is `any`.

```go
type any = interface{}
```

The function has a return type of `[]T` and an input type of `[]T`. Here, type parameter `T` is used to define more types that are used inside the function. These generic functions are instantiated by passing the type value to the type parameter.

```go
reverseInt:= reverse[int]
```

[Playground link](https://go.dev/play/p/gQkKZ651kUu)

(Note: When a type parameter is passed to a type, it is called “instantiated”)

Go’s compiler infers the type parameter by checking the arguments passed to functions. In our first example, it automatically infers that the type parameter is `int`, and often you can skip passing it.

```go
// without passing type
fmt.Println(reverse([]int{1, 2, 3, 4, 5}))

// passing type
fmt.Println(reverse[int]([]int{1, 2, 3, 4, 5}))
```


## Type parameters

As you have seen in the above snippets, generics allows for reducing boilerplate code by providing a solution to represent code with actual types. Any number of type parameters can be passed to a function or struct.

### Type parameters in functions

Using type parameters in functions allows programmers to write code generics over types.

The compiler will create a separate definition for each combination of types passed at instantiation or create an interface-based definition derived from usage patterns and some other conditions which are out of the scope of this article.

// Here T is type parameter, it work similiar to type
func print[T any](v T){
 fmt.Println(v)
}

[Playground link](https://go.dev/play/p/fj2GkvJofWF)

### Type parameters in special types

Generics is very useful with special types, as it allows us to write utility functions over special types.

#### **Slice**

When creating a slice, only one type is required, so only one type parameter is necessary. The example below shows the usage for type parameter `T` with a slice.

```go
// ForEach on slice, that will execute a function on each element of slice.
func ForEach[T any](s []T, f func(ele T, i int , s []T)){
    for i,ele := range s {
        f(ele,i,s)
    }
}
```


[Playground link](https://go.dev/play/p/o6HmleC9Hqx)

#### **Map**

The map requires two types, a `key` type and a `value` type. The value type doesn’t have any restraints but the key type should always satisfy the `comparable` constraint.

```go
// keys return the key of a map
// here m is generic using K and V
// V is contraint using any
// K is restrained using comparable i.e any type that supports != and == operation
func keys[K comparable, V any](m map[K]V) []K {
// creating a slice of type K with length of map
    key := make([]K, len(m))
    i := 0
    for k, _ := range m {
        key[i] = k
        i++
    }
    return key
}
```


Similarly, channels are also supported by generics.

### Type parameters in structs

Go allows defining `structs` with a type parameter. The syntax is similar to the generic function. The type parameter is usable in the method and data members on the struct.

```go
// T is type parameter here, with any constraint
type MyStruct[T any] struct {
    inner T
}

// No new type parameter is allowed in struct methods
func (m *MyStruct[T]) Get() T {
    return m.inner
}
func (m *MyStruct[T]) Set(v T) {
    m.inner = v
}
```


Defining new type parameters is not allowed in struct methods, but type parameters defined in struct definitions are usable in methods.

### Type parameters in generic types

Generic types can be nested within other types. The type parameter defined in a function or struct can be passed to any other type with type parameters.

```go
// Generic struct with two generic types
type Enteries[K, V any] struct {
    Key   K
    Value V
}

// since map needs comparable constraint on key of map K is constraint by comparable
// Here a nested type parameter is used
// Enteries[K,V] intialize a new type and used here as return type
// retrun type of this function is slice of Enteries with K,V type passed
func enteries[K comparable, V any](m map[K]V) []*Enteries[K, V] {
    // define a slice with Enteries type passing K, V type parameters
    e := make([]*Enteries[K, V], len(m))
    i := 0
    for k, v := range m {
        // creating value using new keyword
        newEntery := new(Enteries[K, V])
        newEntery.Key = k
        newEntery.Value = v
        e[i] = newEntery
        i++
    }
    return e
}
```


[Playground link](https://go.dev/play/p/EDoYMtkdfgD)

// here Enteries type is instantiated by providing required type that are defined in enteries function
func enteries[K comparable, V any](m map[K]V) []*Enteries[K, V]

## Type constraints

Unlike generics in C++, Go generics are only allowed to perform specific operations listed in an interface, this interface is known as a constraint.

A constraint is used by the compiler to make sure that the type provided for the function supports all the operations performed by values instantiated using the type parameter.

For example, in the below snippet, any value of type parameter `T` only supports the `String` method — you can use `len()` or any other operation over it.

```go
// Stringer is a constraint
type Stringer interface {
 String() string
}

// Here T has to implement Stringer, T can only perform operations defined by Stringer
func stringer[T Stringer](s T) string {
 return s.String()
}
```


[Playground link](https://go.dev/play/p/M1B_R61QdFA)

### Predefined types in constraints

New additions to Go allow predefined types like `int` and `string` to implement interfaces that are used in constraints. These interfaces with predefined types can only be used as a constraint.

```go
type Number {
  int
}
```


In earlier versions of the Go compiler, predefined types never implemented any interface other than `interface{}`, since there was no method over these types.

A constraint with a predefined type and method can’t be used, since predefined types have no methods on these defined types; it is therefore impossible to implement these constraints.

```go
type Number {
  int
  Name()string // int don't have Name method
}
```


`|` operator will allow a union of types (i.e., multiple concrete types can implement the single interface and the resulting interface allows for common operations in all union types).

```go
type Number interface {
        int | int8 | int16 | int32 | int64 | float32 | float64
}
```

In the above example, the `Number` interface now supports all the operations which are common in provided type, like `<`,`>`, and `+` — all the algorithmic operations are supported by the `Number` interface.

```go
// T as a type param now supports every int,float type
// To able to perform these operation the constrain should be only implementing types that support arthemtic operations
func Min[T Number](x, y T) T {
        if x < y {
                return x
        }
        return y
}
```


[Playground link](https://go.dev/play/p/AMLHjMFJuPR)

Using a union of multiple types allows the performing of common operations supported by these types and writing code that will work for all types in union.

## Type approximation

Go allows creating user-defined types from predefined types like `int`, `string`, etc. `~` operators allow us to specify that interface also supports types with the same underlying types.

For example, if you want to add support for the type `Point` with the underlining type `int` to `Min` function; this is possible using `~`.

```go
// Any Type with given underlying type will be supported by this interface
type Number interface {
        ~int | ~int8 | ~int16 | ~int32 | ~int64 | ~float32 | ~float64
}

// Type with underlying int
type Point int

func Min[T Number](x, y T) T {
        if x < y {
                return x
        }
        return y
}

func main() {
        // creating Point type
        x, y := Point(5), Point(2)

        fmt.Println(Min(x, y))

}
```


[Playground link](https://go.dev/play/p/5XOkNHZ_9_D)

All predefined types support this approximated type — the `~` operator only works with constraints.

```go
// Union operator and type approximation both use together without interface
func Min[T ~int | ~float32 | ~float64](x, y T) T {
        if x < y {
                return x
        }
        return y
}
```


[Playground link](https://go.dev/play/p/w8lEJn933zL)

Constraints also support nesting; the `Number` constraint can be built from the `Integer` constraint and `Float` constraint.

```go
// Integer is made up of all the int types
type Integer interface {
        ~int | ~int8 | ~int16 | ~int32 | ~int64
}

// Float is made up of all the float type
type Float interface {
        ~float32 | ~float64
}

// Number is build from Integer and Float
type Number interface {
        Integer | Float
}

// Using Number
func Min[T Number](x, y T) T {
        if x < y {
                return x
        }
        return y
}
```


[Playground link](https://go.dev/play/p/l5A8ENUhCb9)

## `constraints` package

A new package with a collection of useful constraints has been provided by the Go team — this package contains constraints for `Integer`, `Float` etc.

This package exports constraints for predefined types. Since new predefined types can be added to language, it is better to use constraints defined in the `constraints` package. The most important one of these is the `[Ordered](https://pkg.go.dev/golang.org/x/exp/constraints#Ordered)` constraint. It defines all the types that support `>`,`<`,`==`, and `!=` operators.

```go
func min[T constraints.Ordered](x, y T) T {
    if x > y {
        return x
    } else {
        return y
    }
}
```


[Playground link](https://go.dev/play/p/TDt_NcpypY6?v=gotip)

## Interfaces vs. generics

Generics are not a replacement for interfaces. Generics are designed to work with interfaces and make Go more type-safe, and can also be used to eliminate code repetition.

The interface represents a set of the type that implements the interface, whereas generics are a placeholder for actual types. During compilation, generic code might be turned into an interface-based implementation.

## Conclusion

This article covers how to define a type parameter and how to use a type parameter with existing constructs like functions and structs.

We also looked at union operators and new syntax for implementing an interface for predefined type, as well as using type approximation and using generics with special types like structs.

Once you have all the basic knowledge with a strong foundation, you can dive deeper into more advanced topics; like using generics with [type assertions](https://blog.logrocket.com/type-assertions-vs-type-conversions-go/).

Generics will serve as the building blocks for a great library similar to `lodash` from the JavaScript ecosystem. Generics also help in writing utility functions for Map, Slice, and Channel because it is difficult to write functions that support every type without the `reflect` package.

[Here](https://github.com/anshulrgoyal/go-generic-examples) are some code samples I have written or collected from the original drafts for generics for your convenience.