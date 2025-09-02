https://dev.to/iamismile/understanding-scope-in-go-1m61

Scope is one of the most important foundational concepts in Go (and any programming language). If you understand it well, it will help you write clearer code, avoid subtle bugs, and even master more advanced topics like closures and concurrency.

Let’s explore what scope is, how it works in Go, and how you can remember it, using a simple office building analogy.

---

## [](https://dev.to/iamismile/understanding-scope-in-go-1m61#what-is-scope)🧐 What is Scope?

Scope refers to **where a variable can be seen and used in your code**. It's like a permission system: who can read a variable depends on where it was declared.

**📌 Think of it like a note you wrote**  
Not everyone can see your note. It depends on **where you left it**.

- If you tape it to your office’s main wall, everyone in the department can see it.
- If you hide it in your desk drawer, only you can read it.

This "visibility boundary" is what scope means in programming.

---

## [](https://dev.to/iamismile/understanding-scope-in-go-1m61#the-company-building-analogy)🏢 The Company Building Analogy

Imagine your Go program as a **company office building**. Each part of the building has different levels of access and visibility.

|Real-World Office Element|Go Code Equivalent|
|---|---|
|🗂️ Departments (e.g. HR, IT)|Packages|
|🛋️ Meeting rooms inside departments|Functions|
|🗄️ Drawers inside meeting rooms|Code blocks (`if`, `for`)|
|📝 Notes lying around|Variables|

**🧱 Scope Hierarchy (Top → Bottom)**  

```
🏢 Package Level (Department Wall Notes)
└── 🛋️ Function Level (Meeting Room Table Notes)
    └── 🗄️ Block Level (Drawer Notes)
```

**📌 Rule of Thumb**: You can always read notes from **above**, but not **beside** or **below**.

- If you're in a drawer (block), you can read the table (function) and wall (package) notes.
- But if you're on the department floor (package level), you cannot see inside someone's drawer.

---

## [](https://dev.to/iamismile/understanding-scope-in-go-1m61#what-is-lexical-scope)🧭 What is Lexical Scope?

Go uses **lexical scope** (also called **static scope**), which means:

- Scope is determined **by the position of code**, not by how the program runs.
- **Inner scopes can access outer variables**, but outer scopes can't see inside.
- The visibility of variables is decided at **compile time**, not runtime.

**📌 In our analogy:** If you're in a drawer, you can see notes on the meeting room table and department wall, but someone on the department floor can't see notes inside someone's drawer.

---

### [](https://dev.to/iamismile/understanding-scope-in-go-1m61#basic-scope-example)📝 Basic Scope Example

```go
package main

import "fmt"

var outer = "I'm at package level" // 🏢 Department wall note

func show() {
    var inner = "I'm at function level" // 🛋️ Meeting room table note
    fmt.Println(outer) // ✅ Can read wall notes
    fmt.Println(inner) // ✅ Can read table notes
}

func main() {
    show()
    fmt.Println(outer) // ✅ Can still read wall notes
    // fmt.Println(inner) // ❌ Error: can't read inside a meeting room
}
```

---

### [](https://dev.to/iamismile/understanding-scope-in-go-1m61#block-scope-in-action)🗄️ Block Scope in Action

Variables declared in blocks like `if` and `for` are only visible inside those blocks.  

```go
package main

import "fmt"

func example() {
    x := 1 // 🛋️ Table note

    if true {
        y := 2            // 🗄️ Drawer note
        x = 10            // ✅ Can modify outer table note
        fmt.Println(x, y) // ✅ Can read both: 10, 2
    }

    fmt.Println(x) // ✅ Still 10
    // fmt.Println(y) // ❌ Error: y is not accessible here
}

func main() {
    example()
}
```

**Loops Also Create Block Scopes**  

```go
package main

import "fmt"

func loops() {
    for i := 0; i < 3; i++ {
        message := fmt.Sprintf("Loop %d", i) // 🗄️ Drawer note
        fmt.Println(message)
    }

    // fmt.Println(i)       // ❌ Not accessible
    // fmt.Println(message) // ❌ Not accessible
}

func main() {
    loops()
}
```

---

### [](https://dev.to/iamismile/understanding-scope-in-go-1m61#variable-shadowing-the-impostor-notes)👥 Variable Shadowing: The Impostor Notes

Shadowing occurs when you declare a variable with the same name in a nested scope. It temporarily hides the outer one.  

```go
package main

import "fmt"

var name = "Global Alice" // 🏢 Wall note

func meeting() {
    var name = "Local Bob" // 🛋️ Table note (shadows global)
    fmt.Println("In meeting:", name) // "Local Bob"

    if true {
        var name = "Block Charlie" // 🗄️ Drawer note (shadows Bob)
        fmt.Println("In block:", name) // "Block Charlie"
    }

    fmt.Println("Back in meeting:", name) // "Local Bob"
}

func main() {
    fmt.Println("Global:", name)
    meeting()
    fmt.Println("Still global:", name)
}
```

**🔍 Note**: Each `var name = ...` is a new variable, not a reassignment. It creates a shadow, not an override.

**🚨 Warning**: Shadowing can confuse readers and introduce subtle bugs. Use distinct variable names where clarity matters.

---

### [](https://dev.to/iamismile/understanding-scope-in-go-1m61#package-scope-public-vs-private)🔒 Package Scope: Public vs Private

In Go, **capitalization determines visibility across packages**:

- **lowercase** → private to the package (🏢 department-only notes)
- **Uppercase** → exported, visible to other packages (📢 public announcement)

**Example: Package Structure**  

```go
// File: mypackage/data.go
package mypackage

var secretKey = "hidden"    // 🔒 Only visible in mypackage
var PublicData = "everyone" // 📢 Visible to other packages

func privateFunc() {        // 🔒 Private
    // Only callable in this package
}

func PublicFunc() {         // 📢 Exported
    // Callable from outside
}
```

**Cross-Package Access Example**  

```go
// File: main.go
package main

import (
    "fmt"
    "myproject/mypackage"
)

func main() {
    fmt.Println(mypackage.PublicData) // ✅ Works
    mypackage.PublicFunc()            // ✅ Works

    // fmt.Println(mypackage.secretKey)  // ❌ Compile error
    // mypackage.privateFunc()           // ❌ Compile error
}
```

---

## [](https://dev.to/iamismile/understanding-scope-in-go-1m61#a-note-on-closures-and-scope)🧠 A Note on Closures and Scope

Closures are functions that **capture** variables from their surrounding scope, even after the outer function has exited.  

```go
package main

import "fmt"

func makeCounter() func() int {
    count := 0  // 🛋️ Table note that's "captured"
    return func() int {
        count++ // Modifies the captured variable
        return count
    }
}

func main() {
    counter1 := makeCounter()
    counter2 := makeCounter()

    fmt.Println(counter1()) // 1
    fmt.Println(counter1()) // 2
    fmt.Println(counter2()) // 1 (separate instance)
    fmt.Println(counter1()) // 3
}
```

> 🔁 Each returned function remembers the scope it was created in, even if that scope is "gone" in the normal program flow.

**💡 Why This Works**  
Go **moves captured variables like `count` to the heap**, so they live beyond the lifetime of the outer function. That’s why `counter1()` and `counter2()` don’t interfere with each other.

---

## [](https://dev.to/iamismile/understanding-scope-in-go-1m61#key-takeaways)🎯 Key Takeaways

- **Scope defines where a variable can be accessed** - think of it as visibility boundaries in your office building.
- **Go uses lexical (static) scope** - it's based on where you write code, not how the program runs.
- **Inner scopes can access outer variables, but not the other way around** - you can read notes from above, but not below.
- **Variable shadowing creates new variables with the same name** - be careful not to accidentally shadow when you meant to reassign.
- **Closures capture variables from their surrounding scope** - even after the outer function ends, the captured variables remain accessible.
- **Package capitalization determines cross-package visibility** - lowercase for private, Uppercase for exported.