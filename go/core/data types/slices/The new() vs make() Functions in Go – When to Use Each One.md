https://www.freecodecamp.org/news/new-vs-make-functions-in-go/

Go, also known as Golang, is a statically-typed, compiled programming language designed for simplicity and efficiency.

When it comes to working with data structures like slices, maps, and channels, you'll likely encounter the `new()` and `make()` functions. While both are used for memory allocation, they serve distinct purposes.

In this article, we'll explore the differences between `new()` and `make()` in Go and discuss when to use each.

## The `new()` Function

The `new()` function in Go is a built-in function that allocates memory for a new zeroed value of a specified type and returns a pointer to it. It is primarily used for initializing and obtaining a pointer to a newly allocated zeroed value of a given type, usually for data types like structs.

Here's a simple example:

```go
package main

import "fmt"

type Person struct {
    Name     string
    Age      int
    Gender     string
}

func main() {
    // Using new() to allocate memory for a Person struct
    p := new(Person)

    // Initializing the fields
    p.Name = "John Doe"
    p.Age = 30
    p.Gender = "Male"

    fmt.Println(p)
}
```

In this example, `new(Person)` allocates memory for a new `Person` struct, and `p` is a pointer to the newly allocated zeroed value.

## The `make()` Function

On the other hand, the `make()` function is used for initializing slices, maps, and channels – data structures that require runtime initialization. Unlike `new()`, `make()` returns an initialized (non-zeroed) value of a specified type.

Let's look at an example using a slice:

```go
package main

import "fmt"

func main() {
    // Using make() to create a slice with a specified length and capacity
    s := make([]int, 10, 15)

    // Initializing the elements
    for i := 0; i < 10; i++ {
        s[i] = i + 1
    }

    fmt.Println(s)
}
```

In this example, `make([]int, 10, 15)` creates a slice of integers with a length of 10 and a capacity of 15. The `make()` function ensures that the slice is initialized with non-zero values.

## When to Use `new()` and `make()` in Go

### Use `new()` for Value Types

When dealing with value types like structs, you can use `new()` to allocate memory for a new zeroed value. This is suitable for scenarios where you want a pointer to an initialized structure.

```go
p := new(Person)
```

### Use `make()` for Reference Types:

For slices, maps, and channels, where initialization involves setting up data structures and internal pointers, use `make()` to create an initialized instance.

```go
s := make([]int, 5, 10)
```

### Pointer vs. Value:

Keep in mind that `new()` returns a pointer, while `make()` returns a non-zeroed value. Choose the appropriate method based on whether you need a pointer or an initialized value.

## Conclusion

Understanding the distinction between `new()` and `make()` in Go is crucial for writing clean and efficient code. By using the right method for the appropriate data types, you can ensure proper memory allocation and initialization in your Go programs.