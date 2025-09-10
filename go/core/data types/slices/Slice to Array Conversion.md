Convert slice to array using `[N]T(slice)` (Go 1.17+). Copies data from slice to fixed-size array. Panics if slice has fewer than N elements. Useful when array semantics or specific size guarantees are needed.

## Go – Create Slice From Array

In Go, slices can be created from arrays by referencing a portion of the array. Slices are lightweight and dynamic views over an array, allowing you to access and manipulate elements without copying the data. This makes them an efficient way to work with subsets of an array.

In this tutorial, we will explore how to create slices from arrays in Go, demonstrate practical examples, and provide detailed explanations for each example.

---

## Steps to Create a Slice From an Array

- **Declare an Array:** Define an array with the desired size and type.
- **Use Slicing Syntax:** Create a slice by specifying the range of indices using `array[start:end]`.
- **Include Start Index:** The start index is inclusive.
- **Exclude End Index:** The end index is exclusive.

---

## Examples of Creating a Slice From an Array

### 1 Basic Slice Creation

This example demonstrates how to create a slice from an array:

</>

Copy

```go
package main

import "fmt"

func main() {
    // Declare an array
    arr := [5]int{10, 20, 30, 40, 50}

    // Create a slice from the array
    slice := arr[1:4]

    // Print the array and the slice
    fmt.Println("Array:", arr)
    fmt.Println("Slice:", slice)
}
```

#### Explanation

1. **Array Declaration:** The array `arr` is initialized with five integers.
2. **Slice Creation:** The slice `slice` is created using the syntax `arr[1:4]`, which includes elements at indices `1`, `2`, and `3`.
3. **Array and Slice Independence:** The slice references the same underlying array, meaning changes to the slice will reflect in the array and vice versa.

#### Output

![](https://www.tutorialkart.com/wp-content/uploads/2025/01/golang-create-slice-from-array-1.png)

---

### 2 Slice the Entire Array

This example demonstrates how to create a slice that includes all elements of the array:

</>

Copy

```go
package main

import "fmt"

func main() {
    // Declare an array
    arr := [5]int{10, 20, 30, 40, 50}

    // Create a slice of the entire array
    slice := arr[:]

    // Print the array and the slice
    fmt.Println("Array:", arr)
    fmt.Println("Slice:", slice)
}
```

#### Explanation

1. **Array Declaration:** The array `arr` is initialized with five integers.
2. **Slice Creation:** The slice `slice` is created using `arr[:]`, which includes all elements of the array.
3. **Same Underlying Array:** The slice and the array share the same underlying data, making them interdependent.

#### Output

![](https://www.tutorialkart.com/wp-content/uploads/2025/01/golang-create-slice-from-array-2.png)

---

### 3 Modify Slice and Observe Array

This example demonstrates how modifying a slice affects the underlying array:

</>

Copy

```go
package main

import "fmt"

func main() {
    // Declare an array
    arr := [5]int{10, 20, 30, 40, 50}

    // Create a slice from the array
    slice := arr[1:4]

    // Modify the slice
    slice[0] = 99

    // Print the array and the slice
    fmt.Println("Array after modification:", arr)
    fmt.Println("Slice after modification:", slice)
}
```

#### Explanation

1. **Array Declaration:** The array `arr` is initialized with five integers.
2. **Slice Creation:** The slice `slice` is created using `arr[1:4]`, referencing a subset of the array.
3. **Modify Slice:** The first element of the slice is modified to `99`, which updates the corresponding element in the array.
4. **Shared Data:** Since the slice and array share the same underlying data, changes to one reflect in the other.

#### Output

![](https://www.tutorialkart.com/wp-content/uploads/2025/01/golang-create-slice-from-array-3.png)

---

## Points to Remember

- **Shared Data:** Slices created from arrays share the same underlying array, so changes in one affect the other.
- **Index Boundaries:** Ensure the slice indices are within the valid range of the array to avoid runtime errors.
- **Flexible Slicing:** Use slicing syntax to create views of any subset of the array.
- **Efficient Memory Usage:** Slices avoid copying data and are lightweight references to an array.