https://golangprojectstructure.com/default-zero-values-in-go-code/

This post will discuss what a zero value is and how it can be relied upon to give some default state to initialized variables.

It’s important to know the zero values for each of the basic data types in Go, in order to use them in your own code, so we will also provide a list of the most important zero values.

![A handheld electronic calculator, showing an initial value of zero on its display.](https://golangprojectstructure.com/wp-content/uploads/2022/09/zero-value-calculator-default-512x341.jpg)

The initial value displayed on a calculator’s screen is usually zero. Likewise, variables in Go have initial values, and these values are also zero for integer-typed variables.

Finally, we will look at some examples that make use of zero values, enabling us as programmers to write less code, relying on the compiler to set sensible defaults for us.

Table of Contents

- [What Is a Zero Value?](https://golangprojectstructure.com/default-zero-values-in-go-code/#what-is-a-zero-value "What Is a Zero Value?")
- [Listing the Zero Values for Every Basic Data Type](https://golangprojectstructure.com/default-zero-values-in-go-code/#listing-the-zero-values-for-every-basic-data-type "Listing the Zero Values for Every Basic Data Type")
- [Using the Zero Value as a Default](https://golangprojectstructure.com/default-zero-values-in-go-code/#using-the-zero-value-as-a-default "Using the Zero Value as a Default")
- [Working With the Zero Value of Slices](https://golangprojectstructure.com/default-zero-values-in-go-code/#working-with-the-zero-value-of-slices "Working With the Zero Value of Slices")
- [The Zero Value of Complex Types May Be Implicitly Initialized](https://golangprojectstructure.com/default-zero-values-in-go-code/#the-zero-value-of-complex-types-may-be-implicitly-initialized "The Zero Value of Complex Types May Be Implicitly Initialized")
- [Some Complex Types Require Explicit Initialization](https://golangprojectstructure.com/default-zero-values-in-go-code/#some-complex-types-require-explicit-initialization "Some Complex Types Require Explicit Initialization")

## What Is a Zero Value?

When a variable is declared without having an initial value explicitly assigned, then it is automatically assigned the type’s **zero value** by the Go compiler.

This ensures that every variable has a valid value, whether it’s been set by the programmer or not.

For example, every integer type has a zero value of zero — and, perhaps unsurprisingly, that’s where the value’s name comes from. It could just have easily been called a **default value**.

Look at the example code below:

```go
package main

import "fmt"

func main() {
	var number int

	fmt.Printf(
		"The zero value of 'number' is %v.\n",
		number,
	)
}
```

We haven’t assigned an explicit value for the number variable, but the compiler has automatically set it to `0`, as you will see if you run the code.

The `"%v"` verb used in the `fmt.Printf` string displays a value in its default format, but we could have used the `"%d"` verb instead, which is used specifically for integers, since we already knew the type of the `number` variable.

## Listing the Zero Values for Every Basic Data Type

Below is a table containing the zero values for all of the major data types in Go:

|Data Type|Zero Value|
|---|---|
|`int`, `int8`, `int16`, `int32`, `int64`|`0`|
|`uint`, `uint8`, `uint16`, `uint32`, `uint64`|`0`|
|`byte`, `rune`|`0`|
|`float32`, `float64`|`0.0`|
|`string`|`""`|
|`bool`|`false`|
|`chan`, `interface`, `map`, `func`|`nil`|
|pointer, slice, array|`nil`|

Most of the zero values are obvious defaults. It makes sense for integers to start from zero, for instance. It also makes sense for strings to be initialized as empty, with a length of zero, and for `boolean` variables to be false, if not explicitly set to `true`.

However, as you can see from the table below, there are also many data types that have a zero value of `nil`. These types generally have to be explicitly initialized in some way before they really become useful.

The following quotation from Go’s official documentation explains:

> When memory is allocated to store a value, either through a declaration or a call of make or new, and no explicit initialization is provided, the memory is given a default initialization.
> 
> —[Specification for the Go programming language](https://go.dev/ref/spec#The_zero_value)
> 
> [](https://golangprojectstructure.com/default-zero-values-in-go-code/)

## Using the Zero Value as a Default

Whenever you need to rely on a default value for a variable, you should consider whether the zero value will provide it.

If it will, then you can save yourself work by allowing the Go compiler to initialize the variable, rather than having to do it explicitly yourself.

The code below makes use of zero values as implicit defaults:

```go
package main

import "fmt"

func main() {
	const text = "this is an example"
	var containsE, containsZ bool

	for _, r := range text {
		switch r {
		case 'e', 'E':
			containsE = true
		case 'z', 'Z':
			containsZ = true
		}
	}

	fmt.Printf("Found the letter E: %t\n", containsE)
	fmt.Printf("Found the letter Z: %t\n", containsZ)
}
```

In the simple example above, we iterate through the `text` string, to check whether it contains the letters E and Z.

If the string contains the letter E, then the `containsE` variable will be set to `true`. Likewise, if the string contains the letter Z, then the `containsZ` variable will be set to `true`.

These variables are set independently, so they can both be `true`, both be `false`, or in completely different states, depending on the content of our `text` string.

If the string contains neither of the letters that we’re searching for, then both variables will be set to `false`, even though we haven’t explicitly assigned an initial value to these variables. We simply use the zero values as defaults.

## Working With the Zero Value of Slices

As we saw in the table above, the zero value for slices is `nil`. However, when values are appended to a slice, memory is automatically allocated, if necessary, which means that its zero value is ready for use without any initialization.

The example below shows the use of slice immediately after its declaration (without any explicit initialization):

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var words []string

    words = append(words, "ONE")
    words = append(words, "TWO")
    words = append(words, "THREE")
    words = append(words, "FOUR")

    fmt.Println(strings.Join(words, " AND "))
}
```

You can see that we start by declaring a slice of strings with the name `words`. Its value immediately after declaration is `nil`, but as soon as we append values, the slice is allocated a location in memory, large enough to hold the two strings that we append to it.

The length of the words slice will be `4` and the capacity will be greater than or equal to `4`, since the Go compiler may have chosen to allocate more memory than necessary, but it will never refuse to append a value simply because the slice doesn’t yet have enough memory available (unless the operating system throws an error when trying to allocate more memory; for example, if the current machine’s RAM has been entirely used up).

Finally, we join the strings in our slice together, combining them into one long string that we print out to the console.

(If you want more information about adding values to slices, we have explored this topic in detail in a [previous post](https://golangprojectstructure.com/adding-elements-to-a-slice-in-go/).)

## The Zero Value of Complex Types May Be Implicitly Initialized

More complex types, such as those declared as structs in the Go standard library, also have zero values.

Sometimes these types will need to be initialized explicitly in some way, perhaps by making a function call. However, it is generally better if the types are ready to use straight after declaration.

The code below contains an example of the latter:

```go
package main

import (
	"bytes"
	"io"
	"os"
)

func main() {
	var buffer bytes.Buffer

	buffer.Write([]byte("This is an example.\n"))

	io.Copy(os.Stdout, &buffer)
}
```

We start by declaring a `buffer` variable, which can hold an arbitrary length of bytes. Then we write some text — as a byte slice — to it.

We didn’t need to perform any explicit initialization, because the zero value of `bytes.Buffer` is initialized by default.

Finally, we copy the buffer to `os.Stdout`, which is a relatively low-level way of writing some text to the console.

We could have called `fmt.Print(buffer.String())`, which would perhaps have been a more idiomatic way of doing the same job, but this would first convert the bytes in buffer to a string, rather than simply printing them.

## Some Complex Types Require Explicit Initialization

If you are declaring a custom exported type in a package of your own, you should try to ensure that its zero value is usable, in order to make things easier for anyone who works with your code. However, this goal, laudable though it is, will not always prove practical.

Sometimes a variable needs to be explicitly initialized in some way before it can be used, perhaps by calling a function or method and passing in some options for setup.

The code below shows an example of this from the Go standard library:

```go
package main

import (
	"fmt"
	"log"
	"os/exec"
)

func main() {
	var cmd exec.Cmd // nil

	// explicit initialization
	cmd = *exec.Command("echo", "HELLO,", "WORLD!")

	output, err := cmd.Output()
	if err != nil {
		log.Fatalln(err)
	}

	fmt.Print(string(output))
}
```

We call an external program, in this case the Linux Bash builtin command `echo`, in order to display a message to the console.

It is necessary to create the `cmd` variable to store information about the external command we want to call. However, we do have to call the `exec.Command` function to provide `cmd` with this information.

The first argument to `exec.Command` is the name of the external command that we want to call. The subsequent arguments are passed to the command itself. We know that `echo` will join any strings it is given with a single whitespace character before displaying them.

Calling the `Output` method actually runs the command and returns its standard output as a byte slice.

If we had removed the line that performs the explicit initialization of the `cmd` variable, the `Output` method would have triggered a `panic` (with the error message `"exec: no command"`), since it would not have had anything to run.

Note also that we could have simplified things by declaring _and_ initializing the `cmd` variable on a single line, like so:

```go
cmd := exec.Command("echo", "HELLO,", "WORLD!")
```

This would, of course, have been a more idiomatic way of writing the code, but I separated the two steps in order to emphasize how we can no longer rely on the variable’s zero value.