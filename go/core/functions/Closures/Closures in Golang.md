https://www.geeksforgeeks.org/go-language/closures-in-golang/

Go language provides a special feature known as an [**anonymous function**](https://www.geeksforgeeks.org/go-language/anonymous-function-in-go-language/). An anonymous function can form a closure. A closure is a special type of anonymous function that references variables declared outside of the function itself. It is similar to accessing global variables which are available before the declaration of the function. **Example:** C
```go
// Golang program to illustrate how
// to create a Closure
package main

import "fmt"

func main() {
    
    // Declaring the variable
    GFG := 0

    // Assigning an anonymous  
    // function to a variable 
    counter := func() int {
       GFG += 1
       return GFG
    }

    fmt.Println(counter())
    fmt.Println(counter())
    
  
}
```
**Output:**
```
1
2
```


**Explanation:** The variable **GFG** was not passed as a parameter to the anonymous function but the function has access to it. In this example, there is a slight problem as any other function which will be defined in the main has access to the global variable **GFG** and it can change it without calling **counter** function. Thus closure also provides another aspect which is **data isolation**.
```go
// Golang program to illustrate how
// to create data isolation
package main

import "fmt"

// newCounter function to 
// isolate global variable
func newCounter() func() int {
    GFG := 0
    return func() int {
      GFG += 1
      return GFG
  }
}
func main() {
    // newCounter function is
    // assigned to a variable
    counter := newCounter()

    // invoke counter
    fmt.Println(counter())
    // invoke counter
    fmt.Println(counter())
    
  
}
```
**Output:**

```
1
2
```


**Explanation:** The closure references the variable **GFG** even after the **newCounter()** function has finished running but no other code outside of the **newCounter()** function has access to this variable. This is how data persistency is maintained between function calls while also isolating the data from other code.