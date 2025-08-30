https://www.scaler.com/topics/golang/golang-zero-values/
## Introduction

Variables are the nouns of a programming language; all variables in Golang have a data type. A variable's data type determines the values that the variable can hold and the operations that can be performed on that variable.

In Golang var keyword is used to declare variables. Below is the syntax to declare a variable in Golang.

```go
var name data_type = expression
```

## Zero Values

In **Golang**, When we create a variable with the help of either a normal var keyword or by using new or make keywords without any explicit initialization of a variable, We allocate only memory for such variable, and values of such types are automatically initialized with their zeroed value. Zero values act as default values when we don't initialize the variable.

## Importance of Zero Values

It's not always necessary to assign a value when a field is declared. Fields that are declared but not initialized will be set to a reasonable Zero value by Golang. This Property of setting the values of variables to Zero values is critical for the safety of your program. This makes the life of a developer easy as we don't need to think about initializing the variable that we don't want to.

Let's take an example of an integer array, If we don't have the concept of zero values, We need to set all values just to satisfy the compiler and for the normal working of an array. This explains to us how it's very critical for us that default values are assigned to variables in the background so that we can focus on writing the code.

## Zero Value of Data Types

Below are Zero values of different data types in Golang.

|Type|Zero Value|
|---|---|
|Integer|0|
|Float|0.0|
|Complex|0 Real and 0 Imaginary|
|Byte|0|
|Rune|0|
|String|""|
|Bool|false|
|Map|nil|
|Channel|nil|
|Interface|nil|
|Slices|nil|
|Pointer|nil|

## Example

```go
package main

import "fmt"

func main() {

	// Variable declarations
	var val1 int
	var val2 float64
	var val3 string
	var val4 bool
	var val5 rune
	var val6 []int
	var val7 *string

	//Printing Zero values of different data types
	fmt.Println("Zero value for integer types: ", val1)
	fmt.Println("Zero value for float64 types: ", val2)
	fmt.Println("Zero value for string types: ", val3)
	fmt.Println("Zero value for boolean types: ", val4)
	fmt.Println("Zero value for rune types: ", val5)
	fmt.Println("Zero value for slice types: ", val6)
	fmt.Println("Zero value for pointer types: ", val7)

}
```

**Output :**

```plaintext
Zero value for integer types:  0
Zero value for float64 types:  0
Zero value for string types:
Zero value for boolean types:  false
Zero value for rune types:  0
Zero value for slice types:  []
Zero value for pointer types:  <nil>
```

**Zero value example :** [link](https://go.dev/play/p/emQ7j090brY)

## Conclusion

- Explained the concept of Zero values in Golang.
- Implemented examples for Zero values and described their importance of them in Golang.