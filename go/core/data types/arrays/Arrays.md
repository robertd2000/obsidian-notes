Fixed-size sequences of same-type elements. Size is part of the type, so different sizes are different types. Declared with specific length, initialized to zero values. Value types (copied when assigned/passed). Slices are more commonly used due to flexibility. Foundation for understanding Go's type system.

The type `[n]T` is an array of `n` values of type `T`.

The expression

```go
var a [10]int
```


declares a variable `a` as an array of ten integers.

An array's length is part of its type, so arrays cannot be resized. This seems limiting, but don't worry; Go provides a convenient way of working with arrays.