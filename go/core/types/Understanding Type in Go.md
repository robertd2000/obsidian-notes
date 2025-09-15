https://www.ardanlabs.com/blog/2013/07/understanding-type-in-go.html

When I was coding in C/C++ it was imperative to understand type. If you didn’t, you would get into a lot of trouble with both the compiler and running your code. Regardless of the language, type touches every aspect of programming syntax. A good understand of types and pointers is critical to good programming. This post will focus on type.  
  

Take these bytes of memory for starters:  
  

|FFE4|FFE3|FFE2|FFE1|
|---|---|---|---|
|00000000|11001011|01100101|00001010|

  
What is the value of the byte at address FFE1? If you try to answer the question you will be wrong. Why, because I have not told you what that byte represents. I have not given you the type information.  
  
What if I say that same byte represents a number? Your answer would probably be 10 and again you would be wrong. Why, because you are assuming that when I said it was a number I meant a base 10 number.  
  

**Number Bases:**  
  
All numbering systems have a base that they function within. Since you were  
a baby you were taught to count in base 10. This may be due to the fact that  
most of us have 10 fingers and 10 toes. Also, it seems natural to perform  
math in base 10.  
  
Base defines the number of symbols a numbering system contains. In base 10  
there are 10 distinct symbols we use to represent the infinite number of  
things we can count. In base 10 the symbols are 0, 1, 2, 3, 4, 5, 6, 7, 8, 9.  
Once we reach the symbol 9 we need to grow the length of the number. As an  
example, 10, 100 and 1000.  
  
There are two other bases we use all the time in computing. Base 2 or binary  
numbers, such as the bits represented in the diagram above. Base 16 or  
hexadecimal numbers, such as the addresses represented in the diagram above.  
  
In a binary numbering system (base 2), there are only 2 symbols and those  
symbols are 0 and 1.  
  
In a hexadecimal number system (base 16), there are 16 symbols and those  
symbols are 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F.  
  
If there were apples sitting on a table, those apples could be represented  
in any numbering system. We could say there are:  
  
In Base 2:  10010001 apples  
In Base 10: 145 apples  
In Base 16: 91 apples  
  
All of those answers are correct when given the correct base.  
  
Notice the number of symbols required in each numbering system to represent  
those apples. The larger the base, the more efficient the numbering system.  
  
Using base 16 for computer addresses, IP addresses and color codes makes  
a lot of sense now.  
  
Look at the number for the HTML color white in all three bases:  
  
In Base 2:  1111 1111 1111 1111 1111 1111 (24 characters)  
In Base 10: 16777215 (10 characters)  
In Base 16: FFFFFF (6 characters)  
  
Which numbering system would you have chosen to represent colors?

  
Now if I tell you the byte at address FFE1 represents a base 10 number,  your answer of 10 is correct.  
  
Type provides two pieces of information that both the compiler and you need to perform the same exercise we just went through.  
  
1. The amount of memory, in bytes, to look at  
2. The representation of those bytes  
  
The Go language provides these basic numeric types:  
  

**Unsigned Integers**  
uint8, uint16, uint32, uint64  
  
**Signed Integers**  
int8, int16, int32, int64  
  
**Real Numbers**  
float32, float64  
  
**Predeclared Integers**  
uint, int, uintptr

  
The names for these keywords provide both pieces of the type information.  
  
The uint8 contains a base 10 number using one byte of memory. The value can be between 0 to 255.  
  
The int32 contains a base 10 number using 4 bytes of memory. The value can be between -2147483648 to 2147483647.  

  

The predeclared integers get mapped based on the architecture you are building the code against. On a 64 bit OS, int will map to int64 and on a 32 bit OS, it will be mapped to int32.  
  
Everything that is stored in memory comes back to one of these numeric types. Strings in Go are just a series of uint8 types, with rules around stringing those bytes together and identifying end of string positions.  
  
A pointer in Go is of type uintptr. Again, based on the OS architecture this will be a uint32 or uint64. It is good that Go created a special type for pointers. In the old days many C programmers would write code that assumed a pointer value would always fit inside an unsigned int. With upgrades to the language and architectures over time, that eventually was no longer true. A lot of code broke because addresses became larger than the predeclared unsigned int.  
  
Struct types are just combinations of know types that also eventually resolve to a numeric type.  
  
```go
type Example struct{  
    BoolValue bool  
    IntValue  int16  
    FloatValue float32  
}
```


  
This structure represents a complex type. It represents 7 bytes with three different numeric representations. The bool is one byte, the int16 is 2 bytes and the float32 adds 4 more bytes. However, 8 bytes are actually allocated in memory for this struct.  
  
All memory is allocated on an alignment boundary to minimize memory defragmentation. To determine the alignment boundary Go is using for your architecture, you can run the unsafe.Alignof function. The alignment boundary in Go for the 64bit Darwin platform is 8 bytes. So when Go determines the memory allocation for our structs, it will pad bytes to make sure the final memory footprint is a multiple of 8. The compiler will determine where to add the padding.  
  
If you want to learn more about structure member alignment and padding check out this link:  
  
[http://www.geeksforgeeks.org/structure-member-alignment-padding-and-data-packing/](http://www.geeksforgeeks.org/structure-member-alignment-padding-and-data-packing/)  
  
This program shows the padding that Go inserted into the memory footprint for the Example type struct:  
  
```go
package main  
  
import (  
    "fmt"  
    "unsafe"  
)  
  
type Example struct {  
    BoolValue bool  
    IntValue int16  
    FloatValue float32  
}  
  
func main() {  
    example := &Example{  
        BoolValue:  true,  
        IntValue:   10,  
        FloatValue: 3.141592,  
    }  
  
    exampleNext := &Example{  
        BoolValue:  true,  
        IntValue:   10,  
        FloatValue: 3.141592,  
    }  
  
    alignmentBoundary := unsafe.Alignof(example)  
  
    sizeBool := unsafe.Sizeof(example.BoolValue)  
    offsetBool := unsafe.Offsetof(example.BoolValue)  
  
    sizeInt := unsafe.Sizeof(example.IntValue)  
    offsetInt := unsafe.Offsetof(example.IntValue)  
  
    sizeFloat := unsafe.Sizeof(example.FloatValue)  
    offsetFloat := unsafe.Offsetof(example.FloatValue)  
  
    sizeBoolNext := unsafe.Sizeof(exampleNext.BoolValue)  
    offsetBoolNext := unsafe.Offsetof(exampleNext.BoolValue)  
  
    fmt.Printf("Alignment Boundary: %d\n", alignmentBoundary)  
  
    fmt.Printf("BoolValue = Size: %d Offset: %d Addr: %v\n",  
        sizeBool, offsetBool, &example.BoolValue)  
  
    fmt.Printf("IntValue = Size: %d Offset: %d Addr: %v\n",  
        sizeInt, offsetInt, &example.IntValue)  
  
    fmt.Printf("FloatValue = Size: %d Offset: %d Addr: %v\n",  
        sizeFloat, offsetFloat, &example.FloatValue)  
  
    fmt.Printf("Next = Size: %d Offset: %d Addr: %v\n",  
        sizeBoolNext, offsetBoolNext, &exampleNext.BoolValue)  
}
```


  
Here is the output:  
  
```
Alignment Boundary: 8  
BoolValue  = Size: 1  Offset: 0  Addr: 0x21015b018  
IntValue   = Size: 2  Offset: 2  Addr: 0x21015b01a  
FloatValue = Size: 4  Offset: 4  Addr: 0x21015b01c  
Next       = Size: 1  Offset: 0  Addr: 0x21015b020
```


  
The alignment boundary for the type struct is 8 bytes as expected.  
  
The size value shows how much memory, for that field, will be read and written to. As expected, the size is inline with the type information.  
  
The offset value shows how many bytes into the memory footprint we will find the start of that field.  
  
The address is where the start of each field, within the memory footprint, can be found.  
  
We can see that Go is padding 1 byte between the BoolValue and IntValue fields. The offset value and the difference between the two addresses is 2 bytes. You can also see that the next allocation of memory is starting 4 bytes away from the last field in the struct.  
  
Let's prove the 8 byte alignment rule by only keeping the 1 byte bool field in the struct:  
  
```go
package main  
  
import (  
    "fmt"  
    "unsafe"  
)  
  
type Example struct {  
    BoolValue bool  
}  
  
func main() {  
    example := &Example{  
        BoolValue:  true,  
    }  
  
    exampleNext := &Example{  
        BoolValue:  true,  
    }  
  
    alignmentBoundary := unsafe.Alignof(example)  
  
    sizeBool := unsafe.Sizeof(example.BoolValue)  
    offsetBool := unsafe.Offsetof(example.BoolValue)  
  
    sizeBoolNext := unsafe.Sizeof(exampleNext.BoolValue)  
    offsetBoolNext := unsafe.Offsetof(exampleNext.BoolValue)  
  
    fmt.Printf("Alignment Boundary: %d\n", alignmentBoundary)  
  
    fmt.Printf("BoolValue = Size: %d Offset: %d Addr: %v\n",  
        sizeBool, offsetBool, &example.BoolValue)  
  
    fmt.Printf("Next = Size: %d Offset: %d Addr: %v\n",  
        sizeBoolNext, offsetBoolNext, &exampleNext.BoolValue)  
}
```


  
And the output:  
  
```
Alignment Boundary: 8  
BoolValue = Size: 1 Offset: 0 Addr: 0x21015b018  
Next      = Size: 1 Offset: 0 Addr: 0x21015b020
```


  
Subtract the two addresses and you will see there is an 8 byte difference between the two type struct allocations. Also, the next allocation of memory is starting at the same address from the first example. Go is padding 7 bytes to the struct to maintain the alignment boundary.  
  
Regardless of the padding, the value of size truly represents the amount of memory we can read and write to for each field.  
  
We can only manipulate memory when we are working with a numeric type and the assignment operator (=) is how we do it. To make life easier for us, Go has created some complex types that support the assignment operator directly. Some of these types are strings, arrays and slices. To see a complete list of these types check out this document [http://golang.org/ref/spec#Types](http://golang.org/ref/spec#Types).  
  
These complex types abstract the manipulation of the underlining numeric types that can be found in each implementation. In doing so, these complex types can be used like numeric types that directly read and write to memory.  
  
Go is a type safe language. This means that the compiler will always enforce like types on each side of an assignment operator.  This is really important because it prevents us from reading or writing to memory incorrectly.  
  
Imagine if we could do the following. If you try to compile this code you will get an error.  
  
```go
type Example struct{  
    BoolValue bool  
    IntValue  int16  
    FloatValue float32  
}  
  
example := &Example{  
    BoolValue:  true,  
    IntValue:   10,  
    FloatValue: 3.141592,  
}  
  
var pointer *int32  
pointer = *int32(&example.IntValue)  
*pointer = 20
```


  
What I am trying to do is get the memory address of the 2 byte IntValue field and store it in a pointer of type int32. Then I am trying to use the pointer to write a 4 byte integer into that memory address.  If I was able to use that pointer, I would be violating the type rules for the IntValue field and corrupting memory along the way.  
  

|FFE8|FFE7|FFE6|FFE5|FFE4|FFE3|FFE2|FFE1|
|---|---|---|---|---|---|---|---|
|0|0|0|3.14|0|10|0|true|

|pointer|
|---|
|FFE3|

|FFE8|FFE7|FFE6|FFE5|FFE4|FFE3|FFE2|FFE1|
|---|---|---|---|---|---|---|---|
|0|0|0|0|0|20|0|true|

  
Based on the memory footprint above, the pointer would be writing the value of 20 across the 4 bytes between FFE3 and FFE6. The value of IntValue would be 20 as expected but the value of FloatValue would now be 0. Imagine if writing those bytes went outside the memory allocation for this struct and started to corrupt memory in other areas of the application. The bugs that would follow would appear random and unpredictable.  
  
The Go compiler will always make sure assigning memory and casting is safe.  
  
In this casting example the compiler is going to complain:  
  
```go
package main  
  
import (  
    "fmt"  
)  
  
// Create a new type  
type int32Ext int32  
  
func main() {  
    // Cast the number 10 to a value of type Jill  
    var jill int32Ext = 10  
  
    // Assign the value of jill to jack  
    // ** cannot use jill (type int32Ext) as type int32 in assignment **  
    var jack int32 = jill  
  
    // Assign the value of jill to jack by casting  
    // ** the compiler is happy **  
    var jack int32 = int32(jill)  
  
    fmt.Printf("%d\n", jack)  
}
```


  
First we create a new type in the system called int32Ext and tell the compiler this new type represents a single int32. Next we create a new variable called jill and assign the value of 10. The compiler allows the value to be assigned because the numeric type is on the right side of the assignment operator. The compiler knows the assignment is safe.  
  
Now we try to create a second variable called jack of type int32 and assign the jill variable. Here the compiler throws an error:  
  
"cannot use jill (type int32Ext) as type int32 in assignment"  
  
The compiler respects that jill is of type int32Ext and makes no assumptions about the safeness of the assignment.  
  
Now we use casting and the compiler allows the assignment and the value prints as expected. When we perform the cast the compiler checks the safeness of the assignment. In our case it identifies the values are of the same type and allows the assignment.  
  
This may seem like basic stuff to some of you but it is the foundation for working in any programming language. Even if type is abstracted you are manipulating memory and should understand what you are doing.  
  
With this foundation we can talk about pointers in Go and passing parameters to functions next.  
  
As always, I hope this post helps shed some light on things you may not have know.