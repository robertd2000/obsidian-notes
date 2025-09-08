https://www.kelche.co/blog/go/golang-arrays/

In Go, an array is a fixed-length collection of elements of the same type. The elements of an array are stored in contiguous memory locations. Arrays are zero-indexed, meaning the first element of an array is at index 0. The length of an array is part of its type, so arrays of different lengths are considered different types.

**Arrays vs Slices**

Arrays and slices are both used to store multiple values in a single variable. The difference between the two is that arrays have a fixed length, while slices have a dynamic length. Slices are built on top of arrays.

## Creating an Array in Go

Arrays in Go are created using the `var` keyword followed by the array name, the array length, and the type of the elements. The syntax for creating an array is `var arrayName [length]type`.

main.go

```go
...
var arr [4]int
```

Here, we have declared an array named `arr` that will hold 4 integers. The type of the elements is `int` and the array length is 4. The elements of an array are indexed starting from zero.

main.go

```go
...
arr[0] = 1
arr[1] = 2
arr[2] = 3
arr[3] = 4
```

Here we have assigned values to the elements of the array. We can also declare and initialize an array in a single line; this is known as an array literal. As shown below.

main.go

```go
...
arr := [4]int{1, 2, 3, 4}
```

## Accessing Array Elements in Golang

Array elements can be accessed using the index of the element. The index of the first element is 0, and the index of the last element is the length of the array minus 1.

main.go

```go
...
fmt.Println(arr[0]) // 1
fmt.Println(arr[1]) // 2
fmt.Println(arr[2]) // 3
fmt.Println(arr[3]) // 4
```

## Iterating Over an Array in Golang

Array elements can be accessed using a for loop. The index of the first element is 0, and the index of the last element is the length of the array minus 1.

main.go

```go
...
for i := 0; i < len(arr); i++ {
    fmt.Println(arr[i])
}
```

The output of the above code is:

output

```
1
2
3
4
```

We could also use the `range` keyword to iterate over the elements of an array.

main.go

```go
...
for _, v := range arr {
    fmt.Println(v)
}
```

The output of the above code is:

output

```
1
2
3
4
```

The `range` keyword returns both the index and the value of the element. We can use the blank identifier `_` to ignore the index.

## Array Length in Golang

Arrays have the `len` function that returns the length of the array.

main.go

```go
...
fmt.Println(len(arr)) // 4
```

Since arrays are fixed-length, the length of an array cannot be changed after it is created.

## Array Slicing in Golang

Array slicing is a technique to access a subset of an array. The syntax for array slicing is `array[start:end]`. The `start` index is inclusive, and the `end` index is exclusive. The `start` index defaults to 0, and the `end` index defaults to the length of the array.

main.go

```go
...
fmt.Println(arr[0:2]) // [1 2]
fmt.Println(arr[:2]) // [1 2]
fmt.Println(arr[2:]) // [3 4]
fmt.Println(arr[:]) // [1 2 3 4]
```

## Array Comparison in Golang

Arrays can be compared using the `==` operator. The `==` operator returns true if the two arrays have the same length and the same elements in the same order.

main.go

```go
...
arr1 := [4]int{1, 2, 3, 4}
arr2 := [4]int{1, 2, 3, 4}
arr3 := [4]int{1, 2, 3, 5}
fmt.Println(arr1 == arr2) // true
fmt.Println(arr1 == arr3) // false
```

The last print statement returns false because the elements of `arr1` and `arr3` do not contain the same values.

## Array Sorting in Golang

Arrays can be sorted using the `sort` package. The `sort` package provides the `sort.Ints` function that sorts an array of integers in ascending order.

main.go

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    arr := [4]int{4, 3, 2, 1}
    sort.Ints(arr[:])
    fmt.Println(arr) // [1 2 3 4]
}
```

Another tedious way of sorting an array is to use `for` loops. The following code sorts an array of integers in ascending order.

main.go

```go
...
arr := [4]int{4, 3, 2, 1}

for i := 0; i < len(arr); i++ {
    for j := i + 1; j < len(arr); j++ {
        if arr[i] > arr[j] {
            arr[i], arr[j] = arr[j], arr[i]
        }
    }
}

fmt.Println(arr) // [1 2 3 4]
```

## Array Copy in Golang

Arrays can be copied using the `copy` function. The `copy` function takes two arguments: the destination array and the source array. The `copy` function returns the number of elements copied.

main.go

```go
...
arr1 := [4]int{1, 2, 3, 4}
arr2 := [4]int{5, 6, 7, 8}
n := copy(arr2[:], arr1[:])
fmt.Println(arr2) // [1 2 3 4]
fmt.Println(n) // 4
```

## Array Delete in Golang

Elements of an array can be deleted by setting the value of the element to the zero value of the type of the element. The zero value of an integer is 0.

main.go

```go
...
arr := [4]int{1, 2, 3, 4}
arr[1] = 0
fmt.Println(arr) // [1 0 3 4]
```

## Array Contains in Golang

To check if array contains a value, we can use the `sort.Search` function from the `sort` package. The `sort.Search` function takes three arguments: the length of the array, a function that returns true if the element at the given index is greater than or equal to the value, and the value to search for.

main.go

```go
...
arr := [4]int{1, 2, 3, 4}
i := sort.Search(len(arr), func(i int) bool { return arr[i] >= 3 })
if i < len(arr) && arr[i] == 3 {
    fmt.Println("found 3 at index", i) // found 3 at index 2
}
```

The output of the above code is:

output

```
found 3 at index 2
```

## Array Reverse in Golang

Arrays can be reversed in two ways. The first way is to replace the first element with the last element, the second element with the second last element, and so on. The second way is to use the `sort.Reverse` function from the `sort` package.

main.go

```go
...
arr := [4]int{1, 2, 3, 4}
for i, j := 0, len(arr)-1; i < j; i, j = i+1, j-1 {
    arr[i], arr[j] = arr[j], arr[i]
}
fmt.Println(arr) // [4 3 2 1]
```

main.go

```go
...
arr := [4]int{1, 2, 3, 4}
sort.Sort(sort.Reverse(sort.IntSlice(arr[:])))
fmt.Println(arr) // [4 3 2 1]
```

## Multidimensional Arrays in Golang

Arrays can have more than one dimension. A multidimensional array is an array of arrays. The syntax for declaring a multidimensional array is `var arr [rows][columns]type`. The `rows` and `columns` are the number of rows and columns in the multidimensional array, and `type` is the type of the elements in the multidimensional array.

Most of the operations that can be performed on a 1D array can also be performed on a multidimensional array.

### 2D Array in Golang

A 2D array is an array of arrays. The syntax for declaring a 2D array is `var arr [rows][columns]type`. The `rows` and `columns` are the number of rows and columns in the 2D array, and `type` is the type of the elements in the 2D array.

main.go

```go
...
var arr [2][3]int
```

The above code declares a 2D array with 2 rows and 3 columns. The elements of the 2D array are integers.

### Accessing 2D Array Elements in Golang

2D array elements can be accessed using the row and column index of the element. The row index of the first element is 0, and the column index of the first element is 0. The row index of the last element is the number of rows minus 1, and the column index of the last element is the number of columns minus 1.

main.go

```go
...
arr := [2][3]int{{1, 2, 3}, {4, 5, 6}}
fmt.Println(arr[0][0]) // 1
fmt.Println(arr[0][1]) // 2
fmt.Println(arr[0][2]) // 3
fmt.Println(arr[1][0]) // 4
fmt.Println(arr[1][1]) // 5
fmt.Println(arr[1][2]) // 6
```

### 2D Array Length in Golang

The `len` function returns the number of rows in a 2D array. The `len` function does not return the number of columns in a 2D array. However, we can loop through the rows of a 2D array and use the `len` function to get the number of columns in each row.

main.go

```go
...
arr := [2][3]int{{1, 2, 3}, {4, 5, 6}}
fmt.Println(len(arr)) // 2
for i := 0; i < len(arr); i++ {
    fmt.Println(len(arr[i])) // 3 3
}
```

## Array of Slices in Golang

An array of slices is an array where each element is a slice. The syntax for declaring an array of slices is `var arr [size]type`, where `size` is the number of elements in the array, and `type` is the type of the elements in the array.

main.go

```go
...
var arr [2][]int
```

The above code declares an array of slices with 2 elements. The elements of the array are slices of integers.

## Array of Maps in Golang

An array of maps is one in which each element is a map. The syntax for declaring an array of maps is `var arr [size]type`.

main.go

```go
...
var arr [2]map[string]int

arr[0] = map[string]int{"a": 1, "b": 2}
arr[1] = map[string]int{"c": 3, "d": 4}
```

The above code declares an array of maps with 2 elements. The elements of the array are maps with string keys and integer values.

## Array of Functions in Golang

An array of functions is one where each element is a function. The syntax for declaring an array of functions is `var arr [size]type`.

main.go

```go
...
var arr [2]func(int) int

arr[0] = func(x int) int {
    return x * x
}
arr[1] = func(x int) int {
    return x * x * x
}
```

The above code declares an array of functions with 2 elements. The elements of the array are functions that take an integer as an argument and return an integer.

## Array of Interfaces in Golang

An array of interfaces is one where each element is an interface. The syntax for declaring an array of interfaces is `var arr [size]type`.

main.go

```go
...
var arr [2]interface{}

arr[0] = 1
arr[1] = "hello"
```

The above code declares an array of interfaces with 2 elements. The elements of the array are an integer and a string.

## Array of Pointers in Golang

An array of pointers is one where each element is a pointer. The syntax for declaring an array of pointers is `var arr [size]type`.

main.go

```go
...
var arr [2]*int

a := 1
b := 2
arr[0] = &a
arr[1] = &b
```

The above code declares an array of pointers with 2 elements. The elements of the array are pointers to integers.

## Array of Channels in Golang

An array of channels is one where each element is a channel. The syntax for declaring an array of channels is `var arr [size]type`.

main.go

```go
...
var arr [2]chan int

arr[0] = make(chan int)
arr[1] = make(chan int)
```

The above code declares an array of channels with 2 elements. The elements of the array are channels of integers.

## Array of Structs in Golang

An array of structs is one where each element is a struct. The syntax for declaring an array of structs is `var arr [size]type`.

main.go

```go
...
type Person struct {
    Name string
    Age  int
}

var arr [2]Person

arr[0] = Person{Name: "John", Age: 20}
arr[1] = Person{Name: "Jane", Age: 21}
```

The above code declares an array of structs with 2 elements. The elements of the array are `Person` structs.

Thats all about arrays in Golang. I hope you found this guide helpful. If you have any questions or feedback, feel free to reach out to me.

