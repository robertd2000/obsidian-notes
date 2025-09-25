https://simonklee.dk/type-constraints

Generics let you write data structures and functions with types that are specified later. Functions and types can have type parameters, and instantiated with type arguments, at compile time. Because this occurs at compile time, type parameters — have type constraints. Type constraints restrict the set of allowed type arguments.

Type parameters are enclosed in square brackets, have an identifier and set of constraints. Each named type parameter acts as a place holder that is replaced with a type argument upon instantiation. The _type_ of the type parameter is its type constraint. A type constraint is an interfaces that defines a set of allowed types for the respective parameter.

## Without type parameters

First we define `max` without type parameters

```go
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

To call it with `int` as argument types

```go
max(int(1), int(2))
```

And a compile error if we change the argument type to an `uint` for instance.

```go
max(uint(1), uint(2))
```

> cannot use uint(1) (constant 1 of type uint) as type int in argument to ma

## With type parameters

We can change the definition of `max` to use type parameters

```go
func max[T int](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

And we can call it with an `int`. However, the compiler still complains if we call it with an `uint`.

> uint does not implement int

## Set of type constraints

So we define a set of [type constraints](https://go.dev/ref/spec#Type_constraints) that satisfy our requirement.

```go
func max[T int|uint](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

We can now call `max` with an `int` and `uint`.

```go
max(int(1), int(2))
max(uint(1), uint(2))
```

## Naming type constraints

To re-use constraints it's more convenient to define them as named type constraints

```go
type Integer interface {
  int | uint
}
```

And then you can simply refer to the constraint by the identifier you gave it in your function and type definition.

```go
func max[T Integer](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

## Señor type constraint

In the spec they write "The type set of a term of the form `~T` is the set of types whose underlying type is `T`."

Effectivly, this means to allow a type such as `type MyInt int` to be accepted where the type constraint says `int`, you must denote it with a tilde `~int`.

So we update the Integer constraint

```go
type Integer interface {
  ~int | ~uint
}
```

To enable us to write

```go
type MyInt int

println(max(MyInt(5), MyInt(1)))
```

## Complete example

Finally, a complete listing of the program

max.go

```go
package main

type Integer interface {
    ~int | ~uint
}

func max[T Integer](a, b T) T {
    if a > b {
        return a
    }
    return b
}

func main() {
    println(max(int(1), int(2)))
    println(max(uint(5), uint(1)))

    type MyInt int

    println(max(MyInt(5), MyInt(1)))
}
```

```
> go run max.go
2
5
5
```

## Resources

- [The Go Programming Language Specification: Type constraints](https://go.dev/ref/spec#Type_constraints)
- [Tutorial: Getting started with generics](https://go.dev/doc/tutorial/generics)
- [When To Use Generics](https://go.dev/blog/when-generics)