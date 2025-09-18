https://golangdocs.com/anonymous-functions-in-golang

There can be functions that don’t need a name in order to be defined. These are anonymous functions. Go has support for first-class functions. In this post, we will see how to use anonymous functions in the Go programming language.

## What are Anonymous Functions?

Anonymous functions are those which don’t need a name to be defined. And they can be defined in the flow of a program and can immediately be invoked.

## Declaring Anonymous Functions

Declaration syntax for anonymous functions is pretty straightforward. It is no different in syntax than a regular function. But it can be defined inside the flow of a function and even invoked immediately.

|                                                                                                         |                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12 | `package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `func(){`<br><br>        `fmt.Println(``"Inside anonymous function"``)`<br><br>    `}`<br><br>`}` |

## Assigning Anonymous Function to a Variable

An anonymous function can be assigned to a variable and be called using the variable name. Here is an example showing an anonymous function assigned to a variable.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `v := func(){`<br><br>        `fmt.Println(``"Inside anonymous function"``)`<br><br>    `}`<br><br>`}`|

## Invoking Anonymous Functions

Anonymous functions, when assigned to a variable, can be called using the variable name. Also, it can be invoked immediately.

### 1. Immediate Invocation of Anonymous Functions

Immediate invocation happens when a function that is declared gets called immediately after declaration. Below is an example of an immediate invocation of an anonymous function.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `func(){`<br><br>        `fmt.Println(``"This function gets invoked immediately"``)`<br><br>    `}()`<br><br>    `// prints "This function gets invoked immediately"`<br><br>`}`|

### 2. Invoking an Anonymous Function by Name

Anonymous functions can be invoked by using the variable name it’s assigned to. To do that use the variable name and add the parenthesis beside it. Here is how it’s done.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `// declare the variable`<br><br>    `f := func(){`<br><br>        `fmt.Println(``"This is an anonymous function"``)`<br><br>    `}`<br><br>    `// with parameters`<br><br>    `g := func(n` `int``) {`<br><br>        `fmt.Println(n)`<br><br>    `}`<br><br>    `// call it`<br><br>    `f()`           `// prints "This is an anonymous function"`<br><br>    `g(``42``)`         `// prints 42`<br><br>`}`|

## Passing Arguments to an Anonymous Function

Anonymous functions can take any number of arguments of any type just like a regular function. Let’s see an example of using different arguments with an anonymous function.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `func(v` `int``){`<br><br>        `fmt.Println(v)`<br><br>    `}(``42``)`  <br><br>    `// prints 42`<br><br>`}`|

## Passing Anonymous Functions as an Argument

Anonymous functions can be passed as an argument of a function as well. Here is an example of an anonymous function passed as an argument.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `g := func(v string){`<br><br>        `fmt.Println(v)`<br><br>    `}`<br><br>    `func(v string, g func(v string)){`<br><br>        `g(v)`<br><br>    `}(``"Hello!"``, g)` <br><br>    `// prints "Hello!"`<br><br>`}`|

## Returning Anonymous Function from a Function

An anonymous function can be returned from a function. Let’s see an example where we will return an anonymous function from another function.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `f1 := func() func(v` `int``) {`<br><br>        `f := func(v` `int``) {`<br><br>            `fmt.Println(v)`<br><br>        `}`<br><br>        `return` `f`<br><br>    `}`<br><br>    `g := f1()`<br><br>    `g(``42``)`   `// prints 42`<br><br>`}`|

## Closures in Anonymous Function

Go functions support closures. Closures happen when a function references a variable outside of its scope. Here is an example illustrating the closure.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func f() func() {`<br><br>    `n :=` `0`<br><br>    `return` `func() {`<br><br>        `n++`<br><br>        `fmt.Println(n)`<br><br>    `}`<br><br>`}`<br><br>`func main() {`<br><br>    `h0 := f()`<br><br>    `h1 := f()`  <br><br>    `h0()`  `// 1`<br><br>    `h1()`  `// 1` <br><br>    `h0()`  `// 2`<br><br>    `h1()`  `// 2`<br><br>`}`|

Post navigation