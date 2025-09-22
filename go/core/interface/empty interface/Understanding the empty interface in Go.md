https://dev.to/flrnd/understanding-the-empty-interface-in-go-4652

First of all, to know more about interfaces, I suggest reading [How to use interfaces in go](https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go) and [Go Data Structures: Interfaces](https://research.swtch.com/interfaces).

# [](https://dev.to/flrnd/understanding-the-empty-interface-in-go-4652#now-whats-with-the-empty-interface)Now, what's with _The empty interface_?

Let's say we are developing some sort of output printing library. This way, we start thinking on how to design our Print function. The function needs to receive some value(s) and then pass the result to the standard output. Let's see this with an example ([go playground link](https://play.golang.org/p/JRFvPPzWaY4)):  

```go
// This function takes a string as a parameter and print it to the standard output
func Println(format string) (int n, err error) {
    return fmt.Fprintln(os.Stdout, format)
}
```

But what happen if we want to print a Number? or a more complex input? Run the example in [go playground](https://play.golang.org/p/36UPWg3ps04):  

```go
package main

import (
    "fmt"
    "os"
)

func Println(format string) (n int, err error) {
    return fmt.Fprintln(os.Stdout, format)
}

func main() {
    Println("Number: ", 10)
}

---
// And this is what happen if we run our code, as we could expect:
prog.go:14:9: too many arguments in call to Println
    have (string, number)
    want (string)

Go build failed.
```

As we expected, our program fails. We are passing two parameters and we defined our function to receive only one parameter of type `string`.

Then, how do we solve our Println function? Should we need to implement every use case for every crazy idea that comes to mind? Thankfully, we do not.

Go already takes care of situations like this one, and this is when the empty interface comes in handy. In fact, this is how `fmt` package functions are implemented.

First, let's check the `empty interface` definition (from [a tour of go](https://tour.golang.org/methods/14)):

> The interface type that specifies zero methods is known as the empty interface:
> 
> `interface{}`  
> An empty interface may hold values of any type. (Every type implements at least zero methods.)
> 
> Empty interfaces are used by code that handles values of unknown type. For example, fmt.Print takes any number of arguments of type interface{}.

Okey, let's take a look into the [fmt package documentation](https://golang.org/pkg/fmt/#Fprintln), this is the specifications for the Fprintln function we used before in our Println:  
`func Fprintln(w io.Writer, a ...interface{}) (n int, err error)`

That `a ...interface{}` in the second argument means that we are receiving from 1 to N of `empty interface` type values (for those familiar with Javascript spread operator). And if we quote again the definition:

`Empty interfaces are used by code that handles values of _unknown type_.`

Okey, _unknown type_, for our print function we really don't care what type is it, do we? We only are worried about writing it out into the standard output.

So, there it is, this way we don't need to worry about implementing every use case, because Go is handling in runtime the conversion of those `unknown types` for us, and now we can do something like this: `Println("Hello ", 10, &v)` without worrying about parameters.

NOTE: I've used the fmt implementation for this explanation because the empty interface is one of the most confusing things I've encountered while I started learning go, and playing with it in go playground helped me to understand how and why to use it.

NOTE 2: this would be our final program:  

```go
package main

import (
    "fmt"
    "os"
)

// This function is a exact implementation of fmt.Println()
func Println(a ...interface{}) (n int, err error) {
    return fmt.Fprintln(os.Stdout, a... )
}

func main() {
       n := 10                 // number
       s := "Hello!"           // string
       r := []rune("もしもし！")// slice of runes

    // Our final Println accepts different types of values
    // and go translate them in run time.
    Println(s, n, r)
}
```

If you got this far without falling asleep, thanks for reading! 🤓 🖖