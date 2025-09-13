https://www.zetcode.com/golang/if-else-keywords/

This tutorial explains how to use the `if` keyword in Go. We'll cover conditional logic basics with practical examples of decision-making.

The if statement executes a block of code only when a specified condition evaluates to true. It's fundamental for controlling program flow.

In Go, `if` can be used with optional `else if` and `else` clauses. It supports initialization statements and doesn't require parentheses around conditions.

## Basic if statement

The simplest form of `if` checks a single condition. This example demonstrates basic conditional execution.

basic_if.go

```go

package main

import "fmt"

func main() {
    age := 20
    
    if age >= 18 {
        fmt.Println("You are an adult")
    }
}

```


The code checks if `age` is 18 or more. The message prints only when the condition is true. The braces are required even for single statements.

## If with else clause

The `else` clause provides alternative execution when the condition is false. This example shows both paths.

if_else.go

```go

package main

import "fmt"

func main() {
    temperature := 25.5
    
    if temperature > 30 {
        fmt.Println("It's hot outside")
    } else {
        fmt.Println("It's not too hot")
    }
}

```


When temperature exceeds 30, the first message prints. Otherwise, the else block executes. Only one branch runs in any given execution.

## If with else if ladder

Multiple conditions can be checked using `else if`. This example demonstrates a multi-way decision structure.

if_elseif.go

```go

package main

import "fmt"

func main() {
    score := 85
    
    if score >= 90 {
        fmt.Println("Grade: A")
    } else if score >= 80 {
        fmt.Println("Grade: B")
    } else if score >= 70 {
        fmt.Println("Grade: C")
    } else {
        fmt.Println("Grade: D or below")
    }
}

```


The conditions are evaluated top to bottom. The first true condition executes its block, skipping the rest. The else clause handles all remaining cases.

## If with initialization statement

Go allows variable initialization before the condition. This example shows the idiomatic error checking pattern.

if_init.go

```go

package main

import (
    "fmt"
    "os"
)

func main() {
    if file, err := os.Open("data.txt"); err != nil {
        fmt.Println("Error opening file:", err)
    } else {
        fmt.Println("File opened successfully")
        file.Close()
    }
}

```


The initialization statement declares `file` and `err`. These variables are scoped to the if-else blocks. This pattern is common in Go.

## Nested if statements

If statements can be nested to create complex logic. This example checks multiple conditions in sequence.

nested_if.go

```go

package main

import "fmt"

func main() {
    age := 25
    hasLicense := true
    
    if age >= 16 {
        if hasLicense {
            fmt.Println("You can drive")
        } else {
            fmt.Println("You need a license")
        }
    } else {
        fmt.Println("You're too young to drive")
    }
}

```


The outer if checks age, while the inner if checks license status. Each condition must be true for the full path to execute. Proper indentation improves readability.

## Logical operators in conditions

Conditions can combine multiple expressions using logical operators. This example demonstrates AND and OR operations.

logical_if.go

```go

package main

import "fmt"

func main() {
    hour := 14
    isWeekend := false
    
    if hour >= 9 && hour <= 17 && !isWeekend {
        fmt.Println("Work hours")
    } else if hour < 7 || hour > 22 {
        fmt.Println("Quiet hours")
    } else {
        fmt.Println("Regular hours")
    }
}

```


The && operator requires all conditions to be true. The || operator needs just one true condition. Parentheses can clarify complex expressions.

## Short-circuit evaluation

Go evaluates logical expressions left to right, stopping early when possible. This example shows how it affects program flow.

short_circuit.go

```go

package main

import "fmt"

func isPositive(n int) bool {
    fmt.Println("Checking positivity...")
    return n > 0
}

func isEven(n int) bool {
    fmt.Println("Checking evenness...")
    return n%2 == 0
}

func main() {
    num := 10
    
    if isPositive(num) && isEven(num) {
        fmt.Println("Number is positive and even")
    }
    
    num = -5
    if isPositive(num) && isEven(num) {
        fmt.Println("This won't print")
    }
}

```


With &&, if the first condition is false, the second isn't evaluated. With ||, if the first is true, the second is skipped. This can prevent unnecessary computations.

## Source

[Go language specification](https://go.dev/ref/spec#If_statements)

This tutorial covered the `if` keyword in Go with practical examples of conditional execution in various scenarios.