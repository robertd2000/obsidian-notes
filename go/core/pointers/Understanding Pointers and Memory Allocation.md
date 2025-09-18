https://www.ardanlabs.com/blog/2013/07/understanding-pointers-and-memory.html

In the documentation provided by the Go language team you will find great information on pointers and memory allocation. Here is a link to that documentation:  
  
[](http://golang.org/doc/faq#Pointers)[http://golang.org/doc/faq#Pointers](http://golang.org/doc/faq#Pointers)  
  
We need to start with the understanding that all variables contain a value. Based on the type that variable represents will determine how we can use it to manipulate the memory it contains. Read this post to learn more: [Understanding Type In Go](https://www.ardanlabs.com/blog/2013/07/understanding-type-in-go.html)

  
  
In Go we can create variables that contain the "value of" the value itself or an address to the value. When the "value of" the variable is an address, the variable is considered a pointer.  
  
In the diagram below we have a variable called myVariable. The "value of" myVariable is the address to a value that was allocated of the same type. myVariable is considered a pointer variable.  
  

![Screen Shot](../../../images/goinggo/Screen+Shot+2013-07-27+at+2.57.16+PM.png)

  

![Screen Shot](../../../images/goinggo/Screen+Shot+2013-07-27+at+3.01.52+PM.png)

  
In the next diagram the "value of" myVariable is the value itself, not a reference to the value.  
  
To access properties of a value we use a selector operator. The selector operator allows us to access a specific field in the value. The syntax is always Value.FieldName, where the period (.) is the selector operator.  
  
In the C programming language we need to use different selector operators depending on the type of variable we are using. If the "value of" the variable is the value, we use a period (.). If the "value of" the variable is an address, we use an arrow (->).  
  
One really nice thing about Go is that you don't need to worry about what type of selector operator to use. In Go we only use the period (.) regardless if the variable is the value or a pointer. The compiler takes care of the underlying details to access the value.  
  
So why is all of this important? It becomes important when we start using functions to abstract and break up logic. Eventually you need to pass variables to these functions and you need to know what you are passing.  
  
In Go, variables are passed to a function by value. That means the "value of" each variable that is specified is copied onto the stack for access by that function. In this example we call a function that is supposed to change the state of a value that is allocated in main.  
  

package main  
  
import (  
    "fmt"  
    "unsafe"  
)  
  
type MyType struct {  
    Value1 int  
    Value2 string  
}  
  
func main() {  
    // Allocate an value of type MyType  
    myValue := MyType{10, "Bill"}  
  
    // Create a pointer to the memory for myValue  
    //  For Display Purposes  
    pointer := unsafe.Pointer(&myValue)  
  
    // Display the address and values  
    fmt.Printf("Addr: %v Value1 : %d Value2: %s\n",  
        pointer,  
        myValue.Value1,  
        myValue.Value2)  
  
    // Change the values of myValue  
    ChangeMyValue(myValue)  
  
    // Display the address and values  
    fmt.Printf("Addr: %v Value1 : %d Value2: %s\n",  
        pointer,  
        myValue.Value1,  
        myValue.Value2)  
}  
  
func ChangeMyValue(myValue MyType) {  
    // Change the values of myValue  
    myValue.Value1 = 20  
    myValue.Value2 = "Jill"  
  
    // Create a pointer to the memory for myValue  
    pointer := unsafe.Pointer(& myValue)  
  
    // Display the address and values  
    fmt.Printf("Addr: %v Value1 : %d Value2: %s\n",  
        pointer,  
        myValue.Value1,  
        myValue.Value2)  
}

  
Here is the output of the program:  
  

Addr: 0x2101bc000 Value1 : 10 Value2: Bill  
**Addr: 0x2101bc040 Value1 : 20 Value2: Jill**  
Addr: 0x2101bc000 Value1 : 10 Value2: Bill

  
So what went wrong? The changes the function made to main's myValue did not stay changed after the function call. The "value of" the myValue variable in main does not contain a reference to the value, it is not a pointer. The "value of" the myValue variable in main is the value. When we pass the "value of" the myValue variable in main to the function, a copy of the value is placed on the stack. The function is altering its own version of the value. Once the function terminates, the stack is popped, and the copy is technically gone. The "value of" the myValue variable in main is never touched.  
  
To fix this we can allocate the memory in a way to get a reference back. Then the "value of" the myValue variable in main will be the address to the new value, a pointer variable. Then we can change the function to accept the "value of" an address to the value.  
  

package main  
  
import (  
    "fmt"  
    "unsafe"  
)  
  
type MyType struct {  
    Value1 int  
    Value2 string  
}  
  
func main() {  
    // Allocate an value of type MyType  
    myValue := **&MyType**{10, "Bill"}  
  
    // Create a pointer to the memory for myValue  
    //  For Display Purposes  
    pointer := unsafe.Pointer(**myValue**)  
  
    // Display the address and values  
    fmt.Printf("Addr: %v Value1 : %d Value2: %s\n",  
        pointer,  
        myValue.Value1,  
        myValue.Value2)  
  
    // Change the values of myValue  
    ChangeMyValue(myValue)  
  
    // Display the address and values  
    fmt.Printf("Addr: %v Value1 : %d Value2: %s\n",  
        pointer,  
        myValue.Value1,  
        myValue.Value2)  
}  
  
func ChangeMyValue(myValue ***MyType**) {  
    // Change the values of myValue  
    myValue.Value1 = 20  
    myValue.Value2 = "Jill"  
  
    // Create a pointer to the memory for myValue  
    pointer := unsafe.Pointer(**myValue**)  
  
    // Display the address and values  
    fmt.Printf("Addr: %v Value1 : %d Value2: %s\n",  
        pointer,  
        myValue.Value1,  
        myValue.Value2)  
}

  
When we use the ampersand (&) operator to allocate the value, a reference is returned. That means the "value of" the myValue variable in main is now a pointer variable, whose value is the address to the newly allocated value. When we pass the "value of" the myValue variable in main to the function, the functions myValue variable now contains the address to the value, not a copy. We now have two pointers pointing to the same value. The myValue variable in main and the myValue variable in the function.  
  
If we run the program again, the function is now working the way we wanted it to. It changes the state of the value allocated in main.  
  

Addr: 0x2101bc000 Value1 : 10 Value2: Bill  
**Addr: 0x2101bc000 Value1 : 20 Value2: Jill**  
Addr: 0x2101bc000 Value1 : 20 Value2: Jill

  
During the function call the value is no longer being copied on the stack, the address to the value is being copied. The function is now referencing the same value, via the local pointer variable, and changing the values.  
  
The Go document titled "Effective Go" has a great section about memory allocations, which includes how arrays, slices and maps work:  
  
[http://golang.org/doc/effective_go.html#allocation_new](http://golang.org/doc/effective_go.html#allocation_new)  
  
Let's talk about the keywords new and make.  
  
The new keyword is used to allocate values of a specified type in memory. The memory allocation is zeroed out. The memory can't be further initialized on the call to new. In other words, you can't specify specific values for properties of the specified type when using new.  
  
If you want to specify values at the time you allocate the value, use a composite literal. They come in two flavors, with or without specifying the field names.  
  

    // Allocate an value of type MyType  
    // Values must be in the correct order  
    myValue := MyType{10, "Bill"}  
  
   
    // Allocate a value of type MyType  
    // Use labeling to specify the values  
    myValue := MyType{  
        Value1: 10,  
        Value2: "Bill",  
    }

  
The make keyword is used to allocate and initialize slices, maps and channels only. Make does not return a reference, it returns the "value of" a data structure that is created and initialized to manipulate the new slice, map or channel. This data structure contains references to other data structures that are used to manipulate the slice, map or channel.  
  
How does passing a map by value to a function work. Look at this sample code:  
  

package main  
  
import (  
    "fmt"  
    "unsafe"  
)  
  
type MyType struct {  
    Value1 int  
    Value2 string  
}  
  
func main() {  
    myMap := make(map[string]string)  
    myMap["Bill"] = "Jill"  
  
    pointer := unsafe.Pointer(&myMap)  
    fmt.Printf("Addr: %v Value : %s\n", pointer, myMap["Bill"])  
  
    ChangeMyMap(myMap)  
    fmt.Printf("Addr: %v Value : %s\n", pointer, myMap["Bill"])  
  
    ChangeMyMapAddr(&myMap)  
    fmt.Printf("Addr: %v Value : %s\n", pointer, myMap["Bill"])  
}  
  
func ChangeMyMap(myMap map[string]string) {  
    myMap["Bill"] = "Joan"  
  
    pointer := unsafe.Pointer(&myMap)  
  
    fmt.Printf("Addr: %v Value : %s\n", pointer, myMap["Bill"])  
}  
  
// Don't Do This, Just For Use In This Article  
func ChangeMyMapAddr(myMapPointer *map[string]string) {  
    (*myMapPointer)["Bill"] = "Jenny"  
  
    pointer := unsafe.Pointer(myMapPointer)  
  
    fmt.Printf("Addr: %v Value : %s\n", pointer, (*myMapPointer)["Bill"])  
}

  
Here is the output for the program:  
  

Addr: 0x21015b018 Value : Jill  
**Addr: 0x21015b028 Value : Joan**  
Addr: 0x21015b018 Value : Joan  
Addr: 0x21015b018 Value : Jenny  
Addr: 0x21015b018 Value : Jenny

  
We make a map and add a single key called "Bill" assigning the value of "Jill". Then we pass the value of the map variable to the ChangeMyMap function. Remember the myMap variable is not a pointer so the "value of" myMap, which is a data structure, is copied onto the stack during the function call. Because the "value of" myMap is a data structure that contains references to the internals of the map, the function can use its copy of the data structure to make changes to the map that will be seen by main after the function call.  
  
If you look at the output you can see that when we pass the map by value, the function has its own copy of the map data structure. You can see changes made to the map are reflected after the function call. In main we display the value of the map key "Bill" after the function call and it has changed.  
  
It is unnecessary but the ChangeMyMapAddr function shows how you could pass and use a reference to the myMap variable in main. Again the Go team has made sure passing the "value of" a map variable can be performed without problems. Notice how we need to dereference the myMapPointer variable when we want to access the map. This is because the Go compiler will not allow us to access the map through a pointer variable directly. Dereferencing a pointer variable is equivalent to having a variable whose value is the value.  
  
I have taken the time to write this post because sometimes it can be confusing as to what the "value of" your variable contains. If the "value of" your variable is a large value, and you pass the "value of" that variable to a function, you will be making a large copy of that variable on the stack. You want to make sure you're passing addresses to your functions unless you have a very special use case.  
  
Maps, slices and channels are different. You can pass these variables by value without any concern. When we pass a map variable to a function, we are copying a data structure not the entire map.  
  
I recommend that you read the [Effective Go](http://golang.org/doc/effective_go.html) documentation. I have read that document several times since I started programming in Go. As I gain more experience I always go back and read this document again. I always pick up on something new that didn't make sense to me before.