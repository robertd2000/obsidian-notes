https://golangbot.com/switch/

### What is a switch statement?[](https://golangbot.com/switch/#what-is-a-switch-statement)

**A switch is a conditional statement that evaluates an expression and compares it against a list of possible matches and executes the corresponding block of code. It can be considered as an idiomatic way of replacing complex `if else` clauses.**

### Example[](https://golangbot.com/switch/#example)

An example program is worth a hundred words. Let’s start with a simple example which will take a finger number as input and outputs the name of that finger :) . For example, 1 is thumb, 2 is index, and so on.

```go
package main

import (
	"fmt"
)

func main() {
	finger := 4
	fmt.Printf("Finger %d is ", finger)
	switch finger {
	case 1:
		fmt.Println("Thumb")
	case 2:
		fmt.Println("Index")
	case 3:
		fmt.Println("Middle")
	case 4:
		fmt.Println("Ring")
	case 5:
		fmt.Println("Pinky")

	}
}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/94ktmJWlUom)

In the above program `switch finger` in line no. 10, compares the value of `finger` with each of the `case` statements. The cases are evaluated from top to bottom and the first case which matches the expression is executed. In this case, `finger` has a value of `4` and hence

```fallback
Finger 4 is Ring
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

is printed.

### Duplicate cases are not allowed[](https://golangbot.com/switch/#duplicate-cases-are-not-allowed)

Duplicate cases with the same constant value are not allowed.

```go
package main

import (
	"fmt"
)

func main() {
	finger := 4
	fmt.Printf("Finger %d is ", finger)
	switch finger {
	case 1:
		fmt.Println("Thumb")
	case 2:
		fmt.Println("Index")
	case 3:
		fmt.Println("Middle")
	case 4:
		fmt.Println("Ring")
	case 4: //duplicate case
		fmt.Println("Another Ring")
	case 5:
		fmt.Println("Pinky")

	}
}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/7qrmR0hdvHH)

Running the above program will result in the following compilation error

```fallback
./prog.go:19:7: duplicate case 4 (constant of type int) in expression switch
./prog.go:17:7: previous case
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

### Default case[](https://golangbot.com/switch/#default-case)

We have only 5 fingers in our hands. What will happen if we input an incorrect finger number? This is where the default case comes into the picture. The default case will be executed when none of the other cases match.

```go
package main

import (
	"fmt"
)

func main() {
	switch finger := 8; finger {
	case 1:
		fmt.Println("Thumb")
	case 2:
		fmt.Println("Index")
	case 3:
		fmt.Println("Middle")
	case 4:
		fmt.Println("Ring")
	case 5:
		fmt.Println("Pinky")
	default: //default case
		fmt.Println("incorrect finger number")
	}
}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/P3zBv7zHITF)

In the above program `finger` is `8` and it does not match any of the cases and hence `incorrect finger number` in the default case is printed. It’s not necessary that `default` should be the last case in a switch statement. It can be present anywhere in the switch.

You might also have noticed a small change in the declaration of `finger`. It is declared in the switch itself. A switch can include an optional statement that is executed before the expression is evaluated. In line no. 8, `finger` is first declared and then used in the expression. The scope of `finger` in this case is limited to the switch block.

### Multiple expressions in case[](https://golangbot.com/switch/#multiple-expressions-in-case)

It is possible to include multiple expressions in a case by separating them with comma.

```go
package main

import (
	"fmt"
)

func main() {
	letter := "i"
	switch letter {
	case "a", "e", "i", "o", "u": //multiple expressions in case
		fmt.Printf("%s is a vowel", letter)
	default:
		fmt.Printf("%s is not a vowel", letter)
	}
}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/fYI-vbKNGfc)

The above program finds whether `letter` is a vowel or not. The code `case "a", "e", "i", "o", "u":` in line no. 10 matches any of the vowels. Since `i` is a vowel, this program prints

```fallback
i is a vowel
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

### Expressionless switch[](https://golangbot.com/switch/#expressionless-switch)

The expression in a switch is optional and it can be omitted. If the expression is omitted, the switch is considered to be `switch true` and each of the `case` expression is evaluated for truth and the corresponding block of code is executed.

```go
package main

import (
	"fmt"
)

func main() {
	hour := 15 // hour in 24 hour format
	// Using switch to determine the work shift
	switch {
	case hour >= 6 && hour < 12:
		fmt.Println("It's the morning shift.")
	case hour >= 12 && hour < 17:
		fmt.Println("It's the afternoon shift.")
	case hour >= 17 && hour < 21:
		fmt.Println("It's the evening shift.")
	case (hour >= 21 && hour <= 24) || (hour >= 0 && hour < 6):
		fmt.Println("It's the night shift.")
	default:
		fmt.Println("Invalid hour.")
	}
}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/Bso-wGY040c)

In the above program, the expression is absent in switch and hence it is considered as true and each of the cases is evaluated. `hour >= 12 && hour < 17` in line no. 13 is `true` and the program prints

```fallback
It's the afternoon shift.
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

This type of switch can be considered as an alternative to multiple `if else` clauses.

### Fallthrough[](https://golangbot.com/switch/#fallthrough)

In Go, the control comes out of the switch statement immediately after a case is executed. A `fallthrough` statement is used to transfer control to the first statement of the case that is present immediately after the case which has been executed.

Let’s write a program to understand fallthrough. Our program will check whether the input number is less than 50, 100, or 200. For instance, if we input 75, the program will print that 75 is less than both 100 and 200. We will achieve this using `fallthrough`.

```go
package main

import (
	"fmt"
)

func number() int {
	num := 15 * 5
	return num
}

func main() {
	switch num := number(); { //num is not a constant
	case num < 50:
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num < 100:
		fmt.Printf("%d is lesser than 100\n", num)
		fallthrough
	case num < 200:
		fmt.Printf("%d is lesser than 200", num)
	}
}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/yt3kMvYlzLR)

Switch and case expressions need not be only constants. They can be evaluated at runtime too. In the program above `num` is initialized to the return value of the function `number()` in line no. 13. The control comes inside the switch and the cases are evaluated. `case num < 100:` in line no. 17 is true and the program prints `75 is lesser than 100`. The next statement is `fallthrough`. When `fallthrough` is encountered the control moves to the first statement of the next case and also prints `75 is lesser than 200`. The output of the program is

```fallback
75 is lesser than 100
75 is lesser than 200
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

_`fallthrough` should be the last statement in a `case`. If it is present somewhere in the middle, the compiler will complain that `fallthrough statement out of place`._

### Fallthrough happens even when the case evaluates to false[](https://golangbot.com/switch/#fallthrough-happens-even-when-the-case-evaluates-to-false)

There is a subtlety to be considered when using `fallthrough`. Fallthrough will happen even when the case evaluates to false.

Please consider the following program.

```go
package main

import (
	"fmt"
)

func main() {
	switch num := 25; { 
	case num < 50:
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num > 100:
		fmt.Printf("%d is greater than 100\n", num)		
	}

}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/u2AN8LhYYMM)

In the above program, `num` is 25 which is less than 50 and hence the case in line no. 9 evaluates to `true`. A `fallthrough` is present in line no. 11. The next case `case num > 100:` in line no. 12 is false since num < 100. But fallthrough doesn’t consider this. Fallthrough will happen even though the case evaluates to `false`.

The program above will print

```fallback
25 is lesser than 50
25 is greater than 100
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

So be sure that you understand what you are doing when using fallthrough.

One more thing is `fallthrough` cannot be used in the last case of a switch since there are no more cases to fallthrough. If `fallthrough` is present in the last case, it will result in the following compilation error `cannot fallthrough final case in switch`

### Breaking switch[](https://golangbot.com/switch/#breaking-switch)

The `break` statement can be used to terminate a switch early before it completes. Let’s just modify the above example to a contrived one to understand how break works.

Let’s add a condition that if `num` is less than 0 then the switch should terminate.

```go
package main

import (
	"fmt"
)

func main() {
	switch num := -5; {
	case num < 50:
		if num < 0 {
			break
		}
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num < 100:
		fmt.Printf("%d is lesser than 100\n", num)
		fallthrough
	case num < 200:
		fmt.Printf("%d is lesser than 200", num)
	}

}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/UHwBXPYLv1B)

In the above program `num` is `-5`. When the control reaches the [if statement](https://golangbot.com/if-statement/) in line no. 10, the condition is satisfied since `num < 0`. The break statement terminates the switch before it completes and the program doesn’t print anything :).

### Breaking the outer for loop[](https://golangbot.com/switch/#breaking-the-outer-for-loop)

When the switch case is inside a [for loop](https://golangbot.com/loops/), there might be a need to terminate the for loop early. This can be done by labeling the for loop and breaking the for loop using that label inside the switch statement. Let’s look at an example.

Let’s write a program to generate a random even number.

We will create an infinite for loop and use a switch case to determine whether the generated random number is even. If it is even, the generated number is printed and the for loop is terminated using its label. The [`Intn`](https://golang.org/pkg/math/rand/#Rand.Intn) function of the `rand` [package](https://golangbot.com/go-packages/) is used to generate non-negative pseudo-random numbers.

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
randloop:
	for {
		switch i := rand.Intn(100); {
		case i%2 == 0:
			fmt.Printf("Generated even number %d", i)
			break randloop
		}
	}

}
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

go

[Run in playground](https://go.dev/play/p/0bLYOgs2TUk)

In the program above, the for loop is labeled `randloop` in line no. 9. A random number is generated between 0 and 99 (100 is not included) using the `Intn` function in line no. 11. If the generated number is even, the loop is broken in line no. 14 using the label.

This program prints,

```fallback
Generated even number 18
```

[](https://golangbot.com/switch/# "Copy Code")[](https://golangbot.com/switch/# "Toggle Line Numbers")

**Please note that if the break statement is used without the label, the switch statement will only be broken and the loop will continue running. So labeling the loop and using it in the break statement inside the switch is necessary to break the outer for loop.**

This brings us to the end of this tutorial. There is one more type of switch called **type switch**. We will look into this when we learn about [interfaces](https://golangbot.com/interfaces-part-1/).

Please share your valuable comments and feedback. Please consider sharing this tutorial tutorial on [twitter](https://twitter.com/intent/tweet?text=Switch%20%20Statement&url=https%3a%2f%2fgolangbot.com%2fswitch%2f&tw_p=tweetbutton) or [LinkedIn](https://golangbot.com/switch/#linkedinshr). Have a good day.