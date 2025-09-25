https://golangdocs.com/interfaces-in-golang

In general programming interfaces are contracts that have a set of functions to be implemented to fulfill that contract. Go is no different. Go has great support for interfaces and they are implemented in an implicit way. They allow **polymorphism in Go**. In this post, we will talk about interfaces, what they are, and how they can be used.

## What is an Interface?

An interface is an abstract concept which enables **polymorphism** in Go. A variable of that interface can hold the value that implements the type. Type assertion is used to get the underlying concrete value as we will see in this post.

## Declaring an interface in Golang

An interface is declared as a type. Here is the declaration that is used to declare an interface.

`**type interfaceName interface{}**`

## Zero-value of an interface

The zero value of an interface is **nil**. That means it holds no value and type. The code below shows that.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func main() {`<br><br>    `var a` `interface``{}`<br><br>    `fmt.Println(a)`    `// <nil>`<br><br>`}`|

## The empty interface in Go

An interface is empty if it has **no functions** at all. An empty interface holds any type. That’s why it is extremely useful in many cases. Below is the declaration of an empty interface.

`**var i interface{}**`

## Implementing an interface in Golang

An interface is implemented when the type has implemented the [functions](https://golangdocs.com/functions-in-golang) of the interface. Here is an example showing how to implement an interface.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29<br><br>30<br><br>31<br><br>32|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`type Person` `interface``{`<br><br>    `greet() string`<br><br>`}`<br><br>`type Human struct{`<br><br>    `Name string`<br><br>`}`<br><br>`func (h *Human)greet() string {`<br><br>    `return` `"Hi, I am "` `+ h.Name`<br><br>`}`<br><br>`func isAPerson(h Person) {`<br><br>    `fmt.Println(h.greet())`<br><br>`}`<br><br>`func main() {`<br><br>    `var a = Human{``"John"``}`<br><br>    `fmt.Println(a.greet())`    `// Hi, I am John`<br><br>    `// below function will only work`<br><br>    `// if a is also a person.`<br><br>    `// Here we can see polymorphism in action.`<br><br>    `isAPerson(&a)`             `// Hi, I am John`     <br><br>`}`|

## Implementing multiple interfaces in Go

Multiple interfaces can be implemented at the same time. If all the functions are all implemented then the type implements all the interfaces. Below the type, the bird type implements both the interfaces by implementing the functions.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29<br><br>30<br><br>31<br><br>32<br><br>33|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>    `"reflect"`<br><br>`)`<br><br>`type Flyer` `interface``{`<br><br>    `fly() string`<br><br>`}`<br><br>`type Walker` `interface``{`<br><br>    `walk() string`<br><br>`}`<br><br>`type Bird struct{`<br><br>    `Name string`<br><br>`}`<br><br>`func (b *Bird)fly() string{`<br><br>    `return` `"Flying..."`<br><br>`}`<br><br>`func (b *Bird)walk() string{`<br><br>    `return` `"Walking..."`<br><br>`}`<br><br>`func main() {`<br><br>    `var b = Bird{``"Chirper"``}`<br><br>    `fmt.Println(b.fly())`     `// Flying...`<br><br>    `fmt.Println(b.walk())`    `// Walking...` <br><br>`}`|

## Composing interfaces together

Interfaces can be composed together. The composition is one of the most important concepts in software development. When multiple interfaces are implemented then the type has performed composition. This is really helpful where polymorphism is needed.

## Values in an interface

Interface values have a concrete value and a dynamic type.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29<br><br>30<br><br>31|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`type Flyer` `interface``{`<br><br>    `fly() string`<br><br>`}`<br><br>`type Walker` `interface``{`<br><br>    `walk() string`<br><br>`}`<br><br>`type Bird struct{`<br><br>    `Name string`<br><br>`}`<br><br>`func (b *Bird)fly() string{`<br><br>    `return` `"Flying..."`<br><br>`}`<br><br>`func (b *Bird)walk() string{`<br><br>    `return` `"Walking..."`<br><br>`}`<br><br>`func main() {`<br><br>    `var b = Bird{``"Chirper"``}`<br><br>        `// %v for value and %T for type`<br><br>    `fmt.Printf(``"%v --> %T"``, b, b)`    `// {Chirper} --> main.Bird`<br><br>`}`|

In the code above chirper is of type Bird but has a concrete value of `**{Chirpir}**`.

## Type assertion using the interface

Type assertion is a way to get the underlying value an interface holds. This means if an interface variable is assigned a [string](https://golangdocs.com/strings-in-golang) then the underlying value it holds is the string. Here is an example showing how to use type assertion using interfaces.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`type B struct{`<br><br>    `s string`<br><br>`}`<br><br>`func main() {`<br><br>    `var i` `interface``{} = B{``"a sample string"``}`<br><br>    `fmt.Println(i.(B))` `// {a sample string}`<br><br>`}`|

## Type switch using an interface

Type switches are an extremely similar control structure like the switch-cases, the only difference is here the interface type is used to switch between different conditions.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func checkType(i` `interface``{}) {`<br><br>    `switch` `i.(type) {`          `// the switch uses the type of the interface`<br><br>    `case` `int``:`<br><br>        `fmt.Println(``"Int"``)`<br><br>    `case` `string:`<br><br>        `fmt.Println(``"String"``)`<br><br>    `default``:`<br><br>        `fmt.Println(``"Other"``)`<br><br>    `}`<br><br>`}`<br><br>`func main() {`<br><br>    `var i` `interface``{} =` `"A string"`<br><br>    `checkType(i)`   `// String`<br><br>`}`|

## Equality of interface values

The interface values can be equal if any of the conditions shown below are true.

- They both are nil.
- They have the same underlying concrete values and the same dynamic type.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func isEqual(i` `interface``{}, j` `interface``{}) {`<br><br>    `if``(i == j) {`<br><br>        `fmt.Println(``"Equal"``)`<br><br>    `}` `else` `{`<br><br>        `fmt.Println(``"Inequal"``)`<br><br>    `}`<br><br>`}`<br><br>`func main() {`<br><br>    `var i` `interface``{}`<br><br>    `var j` `interface``{}`<br><br>    `isEqual(i, j)`   `// Equal`<br><br>    `var a` `interface``{} =` `"A string"`<br><br>    `var b` `interface``{} =` `"A string"`<br><br>    `isEqual(a, b)`   `// Equal`<br><br>`}`|

## Using interfaces with functions

Interfaces can be passed to functions just like any other type. Here is an example showing the usage of the interface with functions. A great advantage when using an interface is that it allows any type of argument as we can see in this code below.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func f(i` `interface``{}) {`<br><br>    `fmt.Printf(``"%T\n"``, i)`<br><br>`}`<br><br>`func main() {`<br><br>    `var a` `interface``{} =` `"a string"`<br><br>    `var c` `int` `=` `42`<br><br>    `f(a)`   `// string`<br><br>    `f(c)`   `// int`<br><br>`}`|

## Uses of an interface

Interfaces are used in Go where polymorphism is needed. In a function where multiple types can be passed an interface can be used. Interfaces allow Go to have polymorphism.

Interfaces are a great feature in Go and should be used wisely.