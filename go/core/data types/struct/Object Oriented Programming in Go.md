https://www.ardanlabs.com/blog/2013/07/object-oriented-programming-in-go.html

Someone asked a question on the forum today on how to gain the benefits of inheritance without embedding. It is really important for everyone to think in terms of Go and not the languages they are leaving behind. I can’t tell you much code I removed from my early Go implementations because it wasn’t necessary. The language designers have years of experience and knowledge. Hindsight is helping to create a language that is fast, lean and really fun to code in.  

  
I consider Go to be a light object oriented programming language. Yes it does have encapsulation and type member functions but it lacks inheritance and therefore traditional polymorphism. For me, inheritance is useless unless you want to implement polymorphism. With the way interfaces are implemented in Go, there is no need for inheritance. Go took the best parts of OOP, left out the rest and gave us a better way to write polymorphic code.  
  
Here is a quick view of OOP in Go. Start with these three structs:  
  
```go

type Animal struct {  
    Name string  
    mean bool  
}  
  
type Cat struct {  
    Basics Animal  
    MeowStrength int  
}  
  
type Dog struct {  
    Animal  
    BarkStrength int  
}

```

  
Here are three structs you would probably see in any OOP example. We have a base struct and two other structs that are specific to the base. The Animal structure contains attributes that all animals share and the other two structs are specific to cats and dogs.  
  
All of the member properties (fields) are public except for mean. The mean field in the Animal struct starts with a lowercase letter. In Go, the case of the first letter for variables, structs, fields, functions, etc. determine the access specification. Use a capital letter and it's public, use a lowercase letter and it's private.  
  
_Note: The concept of public and private access in Go is not exactly true:_  
[_https://www.ardanlabs.com/blog/2014/03/exportedunexported-identifiers-in-go.html_](https://www.ardanlabs.com/blog/2014/03/exportedunexported-identifiers-in-go.html)  
  
Since there is no inheritance in Go, composition is your only choice. The Cat struct has a field called Basics which is of type Animal. The Dog struct is using an un-named struct (embedding) for the Animal type. It's up to you to decide which is better for you and I will show you both implementations.  
  
I want to thank John McLaughlin for his comment about un-named structs!!  
  
To create a member function (method) for both Cat and Dog, the syntax is as follows:  
  
```go

func **(dog *Dog)** MakeNoise() {  
    barkStrength := **dog**.BarkStrength  
  
    if **dog**.mean == true {  
        barkStrength = barkStrength * 5  
    }  
  
    for bark := 0; bark < barkStrength; bark++ {  
        fmt.Printf("BARK ")  
    }  
  
    fmt.Println("")  
}  
  
func **(cat *Cat)** MakeNoise() {  
    meowStrength := **cat**.MeowStrength  
  
    if **cat**.**Basics**.mean == true {  
        meowStrength = meowStrength * 5  
    }  
  
    for meow := 0; meow < meowStrength; meow++ {  
        fmt.Printf("MEOW ")  
    }  
  
    fmt.Println("")  
}

```


  
Before the name of the method we specify a receiver which is a pointer to each type. Now both Cat and Dog have methods called MakeNoise.  
  
Both these methods do the same thing. Each animal speaks in their native tongue based on their bark or meow strength and if they are mean. Silly, but it shows you how to access the referenced object (value).  
  
When using the Dog reference we access the Animal fields directly. With the Cat reference we use the named field called Basics.  
  
So far we have covered encapsulation, composition, access specifications and member functions. All that is left is how to create polymorphic behavior.  
  
We use interfaces to create polymorphic behavior:  
  
```go

type AnimalSounder interface {  
    MakeNoise()  
}  
  
func MakeSomeNoise(animalSounder AnimalSounder) {  
    animalSounder.MakeNoise()  
}

```


  
Here we add an interface and a public function that takes a value of the interface type. Actually the function will take a reference to a value of a type that implements this interface. An interface is not a type that can be instantiated. An interface is a declaration of behavior that is implemented by other types.  
  
There is a Go convention of naming interfaces with the "er" suffix when the interface only contains one method.
  
In Go, any type that implements an interface, via methods, then represents the interface type. In our case both Cat and Dog have implemented the AnimalSounder interface with pointer receivers and therefore are considered to be of type AnimalSounder.  
  
This means that pointers of both Cat and Dog can be passed as parameters to the MakeSomeNoise function. The MakeSomeNoise function implements polymorphic behavior through the AnimalSounder interface.  
  
If you wanted to reduce the duplication of code in the MakeNoise methods of Cat and Dog, you could create a method for the Animal type to handle it:  
  
```go

func (animal *Animal) PerformNoise(strength int, sound string) {  
    if animal.mean == true {  
        strength = strength * 5  
    }  
  
    for voice := 0; voice < strength; voice++ {  
        fmt.Printf("%s ", sound)  
    }  
  
    fmt.Println("")  
}  
  
func (dog *Dog) MakeNoise() {  
    dog.PerformNoise(dog.BarkStrength, "BARK")  
}  
  
func (cat *Cat) MakeNoise() {  
    cat.Basics.PerformNoise(cat.MeowStrength, "MEOW")  
}


```

  
Now the Animal type has a method with the business logic for making noise. The business logic stays within the scope of the type it belongs to. The other cool benefit is we don't need to pass the mean field in as a parameter because it already belongs to the Animal type.  
  
Here is the complete working sample program:  
  
```go

package main  
  
import (  
    "fmt"  
)  
  
type Animal struct {  
    Name string  
    mean bool  
}  
  
type AnimalSounder interface {  
    MakeNoise()  
}  
  
type Dog struct {  
    Animal  
    BarkStrength int  
}  
  
type Cat struct {  
    Basics Animal  
    MeowStrength int  
}  
  
func main() {  
    myDog := &Dog{  
        Animal{  
           "Rover", // Name  
           false,   // mean  
        },  
        2, // BarkStrength  
    }  
  
    myCat := &Cat{  
        Basics: Animal{  
            Name: "Julius",  
            mean: true,  
        },  
        MeowStrength: 3,  
    }  
  
    MakeSomeNoise(myDog)  
    MakeSomeNoise(myCat)  
}  
  
func (animal *Animal) PerformNoise(strength int, sound string) {  
    if animal.mean == true {  
        strength = strength * 5  
    }  
  
    for voice := 0; voice < strength; voice++ {  
        fmt.Printf("%s ", sound)  
    }  
  
    fmt.Println("")  
}  
  
func (dog *Dog) MakeNoise() {  
    dog.PerformNoise(dog.BarkStrength, "BARK")  
}  
  
func (cat *Cat) MakeNoise() {  
    cat.Basics.PerformNoise(cat.MeowStrength, "MEOW")  
}  
  
func MakeSomeNoise(animalSounder AnimalSounder) {  
    animalSounder.MakeNoise()  
}

```


  
Here is the output:  
  
```
BARK BARK  
MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW MEOW
```


  
Someone posted an example on the board about embedding an interface inside of a struct. Here is an example:  
  
```go

package main  
  
import (  
    "fmt"  
)  
  
type HornSounder interface {  
    SoundHorn()  
}  
  
type Vehicle struct {  
    List [2]HornSounder  
}  
  
type Car struct {  
    Sound string  
}  
  
type Bike struct {  
   Sound string  
}  
  
func main() {  
    vehicle := new(Vehicle)  
    vehicle.List[0] = &Car{"BEEP"}  
    vehicle.List[1] = &Bike{"RING"}  
  
    for _, hornSounder := range vehicle.List {  
        hornSounder.SoundHorn()  
    }  
}  
  
func (car *Car) SoundHorn() {  
    fmt.Println(car.Sound)  
}  
  
func (bike *Bike) SoundHorn() {  
    fmt.Println(bike.Sound)  
}  
  
func PressHorn(hornSounder HornSounder) {  
    hornSounder.SoundHorn()  
}

```


  
In this example the Vehicle struct maintains a list of values that implement the HornSounder interface. In main we create a new vehicle and assign a Car and Bike pointer to the list. This assignment is possible because Car and Bike both implement the interface. Then using a simple loop, we use the interface to sound the horn.  
  
Everything you need to implement OOP in your application is there in Go. As I said before, Go took the best parts of OOP, left out the rest and gave us a better way to write polymorphic code.  
  
To learn more on related topics check out these posts:  
  
[https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html](https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html)  
[https://www.ardanlabs.com/blog/2013/07/how-packages-work-in-go-language.html](https://www.ardanlabs.com/blog/2013/07/how-packages-work-in-go-language.html)  
[https://www.ardanlabs.com/blog/2013/07/singleton-design-pattern-in-go.html](https://www.ardanlabs.com/blog/2013/07/singleton-design-pattern-in-go.html)  
  
I hope this small example helps you in your future Go programming.