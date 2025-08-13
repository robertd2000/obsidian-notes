https://www.geeksforgeeks.org/go-language/composition-in-golang/

Composition is a method employed to write re-usable segments of code. It is achieved when objects are made up of other smaller objects with particular behaviors, in other words, Larger objects with a wider functionality are embedded with smaller objects with specific behaviors. The end goal of composition is the same as that of **inheritance**, however, instead of inheriting features from a parent class/object, larger objects in this regard are composed of the other objects and can thereby use their functionality. Since writing effective code in the Go Language revolves majorly around using structs and interfaces, composition is a key takeaway from what the language has to offer. In this article we will discuss the use of the composition of types with structs and interfaces, through type embedding: **Example 1:** _Composition of [Structs](https://www.geeksforgeeks.org/go-language/structures-in-golang/)_

```go

// Golang program to store information
// about games in structs and display them
package main

import "fmt"

// We create a struct details to hold
// generic information about games
type details struct {
    genre       string
    genreRating string
    reviews     string
}

// We create a struct game to hold
// more specific information about
// a particular game
type game struct {

    name  string
    price string
    // We use composition through
    // embedding to add the
    // fields of the details 
    // struct to the game struct
    details
}

// this is a method defined
// on the details struct
func (d details) showDetails() {
    fmt.Println("Genre:", d.genre)
    fmt.Println("Genre Rating:", d.genreRating)
    fmt.Println("Reviews:", d.reviews)
}

// this is a method defined 
// on the game struct
// this method has access 
// to showDetails() as well since
// the game struct is composed
// of the details struct
func (g game) show() {
    fmt.Println("Name: ", g.name)
    fmt.Println("Price:", g.price)
    g.showDetails()
}

func main() {

    // defining a struct 
    // object of Type details
    action := details{"Action","18+", "mostly positive"}
    
    // defining a struct
    // object of Type game
    newGame := game{"XYZ","$125", action}

    newGame.show()
}
```

**Output:**

```
Name:  XYZ
Price: $125
Genre: Action
Genre Rating: 18+
Reviews: mostly positive
```

**Explanation:** In the above code snippet we have created two structs: _details_ and _game_. The struct _details_ comprises of generic information about games. The struct _game_ is a composite struct, which has fields of its own and those of _details_ as well. This composition has been achieved through type embedding as a result of which the first struct becomes a reusable piece of code. It is interesting to note that the methods which have been defined on the struct _details_ is accessible to objects of _Type game_, simply because _game_ is composed of _details_. **Example 2:** _Composition through Embedding in [Interfaces](https://www.geeksforgeeks.org/go-language/interfaces-in-golang/)_ In the Go language interfaces are implicitly implemented. That is to say, if methods, which are defined in an interface, are used on objects such as structs, then the struct is said to implement the interface. An interface can be embedded with other interfaces in order to form composite interfaces. If all the interfaces in a composite interface are implemented, then the composite interface is also said to be implemented by that object.

```go

// Golang Program to implement composite interfaces
package main

import "fmt"

type purchase interface {
    sell()
}

type display interface {
    show()
}

// We use the two previous
// interfaces to form
// The following composite 
// interface through embedding
type salesman interface {
    purchase
    display
}

type game struct {
    name, price    string
    gameCollection []string
}

// We use the game struct to
// implement the interfaces
func (t game) sell() {
    fmt.Println("--------------------------------------")
    fmt.Println("Name:", t.name)
    fmt.Println("Price:", t.price)
    fmt.Println("--------------------------------------")
}
func (t game) show() {
    fmt.Println("The Games available are: ")
    for _, name := range t.gameCollection {
        fmt.Println(name)
    }
    fmt.Println("--------------------------------------")
}
 
// This method takes the composite
// interface as a parameter
// Since the interface is composed
// of purchase and display
// Hence the child methods of those
// interfaces can be accessed here
func shop(s salesman) {
    fmt.Println(s)
    s.sell()
    s.show()
}

func main() {

    collection := []string{"XYZ", 
        "Trial by Code", "Sea of Rubies"}
    game1 := game{"ABC", "$125", collection}
    shop(game1)

}

```

**Output:**

```
{ABC $125 [XYZ Trial by Code Sea of Rubies]}
--------------------------------------
Name: ABC
Price: $125
--------------------------------------
The Games available are: 
XYZ
Trial by Code
Sea of Rubies
--------------------------------------
```

**Explanation:** Firstly, we created 2 interfaces _purchase_ and _display_ with their own method prototypes. Then we embedded them in the interface _salesman_ in order to form a composite struct. This demonstrates the use of the concept of composition in Go Lang. In the method shop() we have implemented the salesman interface by passing an object of Type game in the method. Since this method implements the composite interface we have access to the child methods of the two interfaces we had initially declared. In this way, effective go programming can be carried out through a clean reusable code.