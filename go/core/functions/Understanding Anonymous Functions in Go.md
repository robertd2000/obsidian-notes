https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd

## **What is an Anonymous Function?**

An **anonymous function** is a function **without a name**. Instead of being declared like a traditional named function, it is defined inline and assigned to a variable or executed immediately.

### [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#basic-syntax)**Basic Syntax**

```go
func main() {
    add := func(a, b int) int {
        return a + b
    }
    fmt.Println(add(3, 5)) // Output: 8
}
```

📌 Here, `add` holds an anonymous function that takes two integers and returns their sum.

---

## [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#use-cases-of-anonymous-functions)**Use Cases of Anonymous Functions**

### [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#passing-anonymous-functions-as-arguments)**1️⃣ Passing Anonymous Functions as Arguments**

Since functions can be passed as arguments, anonymous functions are useful for higher-order functions.  

```go
func operate(a, b int, op func(int, int) int) int {
    return op(a, b)
}

func main() {
    result := operate(4, 2, func(x, y int) int {
        return x * y
    })
    fmt.Println("Multiplication:", result) // Output: 8
}
```

✅ **Use Case:** Useful in callback functions or custom processing logic.

---

### [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#using-anonymous-functions-in-goroutines)**2️⃣ Using Anonymous Functions in Goroutines**

Goroutines allow concurrent execution, and anonymous functions are a great way to define short-lived concurrent tasks.  

```go
func main() {
    go func() {
        fmt.Println("Hello from Goroutine!")
    }()
    time.Sleep(time.Second) // Allow Goroutine to execute
}
```

✅ **Use Case:** Running background tasks without defining a named function.

---

### [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#returning-anonymous-functions-closures)**3️⃣ Returning Anonymous Functions (Closures)**

Closures allow an anonymous function to capture variables from its surrounding scope.  

```go
func multiplier(factor int) func(int) int {
    return func(x int) int {
        return x * factor
    }
}

func main() {
    double := multiplier(2)
    fmt.Println(double(5)) // Output: 10
}
```

✅ **Use Case:** Creating **function factories** that retain state.

---

### [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#storing-anonymous-functions-in-a-map)**4️⃣ Storing Anonymous Functions in a Map**

You can store anonymous functions in a map for dynamic execution.  

```go
func main() {
    operations := map[string]func(int, int) int{
        "add": func(a, b int) int { return a + b },
        "sub": func(a, b int) int { return a - b },
    }
    fmt.Println("Addition:", operations["add"](4, 2)) // Output: 6
}
```

✅ **Use Case:** Implementing **dynamic function lookups** or **command dispatch systems**.

---

## [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#what-is-an-iife-immediately-invoked-function-expression)**What is an IIFE (Immediately Invoked Function Expression)?**

An **IIFE (Immediately Invoked Function Expression)** is an anonymous function that runs **immediately after being defined**.  

```go
func main() {
    result := func(a, b int) int {
        return a + b
    }(3, 5) // Function executes immediately

    fmt.Println(result) // Output: 8
}
```

✅ **Use Case:** One-time setup logic, reducing unnecessary variable scope.

---

## [](https://dev.to/abstractmusa/understanding-anonymous-functions-in-go-a-practical-guide-57hd#conclusion)**Conclusion**

Anonymous functions in Go offer flexibility and concise coding, making them a powerful tool for:

- **Higher-order functions**
- **Concurrent execution (Goroutines)**
- **Closures (Retaining state)**
- **Dynamic function lookups**
- **One-time execution (IIFE)**

By understanding and implementing anonymous functions effectively, you can write **cleaner**, **more modular**, and **efficient** Go code.