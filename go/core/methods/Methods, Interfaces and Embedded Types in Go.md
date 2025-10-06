https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html

**Introduction**  
My business partner Ed asked me what would happen if a struct and an embedded field both implemented the same interface. We asked ourselves two questions:  

- Would the compiler throw an error because we now had two implementations of the interface?

- If the compiler accepted the type declaration, how does the compiler determine which implementation to use for interface calls?

We hacked out some code to answer the questions and then I dug into the specification. What we found was really interesting and we believed worth sharing with others who are also learning the language. Once we learned the mechanics behind methods, interfaces, and embedded types, the answers seemed obvious. To get started, let’s talk about what methods are in Go.  
  
**Methods**  
Go has both functions and methods. In Go, a method is a function that is declared with a [receiver](http://golang.org/ref/spec#Method_declarations). A receiver is a value or a pointer of a [named](http://golang.org/ref/spec#Types) or [struct](http://golang.org/ref/spec#Struct_types) type. All the methods for a given type belong to the type’s method set.  
  
Let’s declare a struct type and a method for that type:  
  
```go
type User struct {  
    Name string  
    Email string  
}  
  
func (u User) Notify() error
```
  
First we declare a struct type named User and then we declare a method named Notify with a receiver that accepts a value of type User. To call the Notify method we need a value or pointer of type User:  
  
```go
// Value of type User can be used to call the method  
// with a value receiver.  
bill := User{"Bill", "bill@email.com"}  
bill.Notify()  
  
// Pointer of type User can also be used to call a method  
// with a value receiver.  
jill := &User{"Jill", "jill@email.com"}  
jill.Notify()
```

In the case where we are using a pointer, Go [adjusts](http://golang.org/ref/spec#Calls) and dereferences the pointer so the call can be made. Be aware that when the receiver is not a pointer, the method is operating against a copy of the receiver value.  
  
We can change the Notify method to use a pointer for the receiver:  
  
```go
func (u *User) Notify() error

```
  
Once again, we can call the Notify method like we did before:  
  
```go
// Value of type User can be used to call the method  
// with a pointer receiver.  
bill := User{"Bill", "bill@email.com"}  
bill.Notify()  
  
// Pointer of type User can be used to call the method  
// with a pointer receiver.  
jill := &User{"Jill", "jill@email.com"}  
jill.Notify()
```


  
If you are unsure about when to use a value or a pointer for the receiver, the Go wiki has a great set of [rules](https://code.google.com/p/go-wiki/wiki/CodeReviewComments#Receiver_Type) that you can follow. The Go wiki also contains a paragraph about the conventions the community follows for [naming](https://code.google.com/p/go-wiki/wiki/CodeReviewComments#Receiver_Names) receivers.  
  
**Interfaces**  
[Interfaces](http://golang.org/doc/effective_go.html#interfaces) in Go are special and provide an incredible amount of flexibility and abstraction for our programs. They are a way of specifying that values and pointers of a particular type can behave in a specific way. From a language perspective, an interface is a type that specifies a [method set](http://golang.org/ref/spec#Method_sets) and all the methods for an [interface type](http://golang.org/ref/spec#Interface_types) are considered to be the interface.  
  
Let’s declare an interface in Go:  
  
```go
type Notifier interface {  
    Notify() error  
}
```


  
Here we declare an interface called Notifier with a single method called Notify. It is a [convention](http://golang.org/doc/effective_go.html#interface-names) in Go to name interfaces with an -er suffix when the interface contains only one method. This is not a hard rule but something we should honor, especially when the interface and method name have the same signature and meaning.  
  
We can specify as many methods as we want for our interfaces. In the standard library you will be hard pressed to find many interfaces with more than two methods.  
  
**Implementing Interfaces**  
Go is unique when it comes to how we implement the interfaces we want our types to support. Go does not require us to explicitly state that our types implement an interface. If every method that belongs to an interface’s method set is implemented by our type, then our type is said to implement the interface.  
  
Let’s continue with our example by creating a function that accepts any value or pointer of a type that implements the Notifier interface:  
  
```go
func SendNotification(notify Notifier) error {  
    return notify.Notify()  
}
```


  
The SendNotification function calls the Notify method that is implemented by the value or pointer that is passed into the function. This function can be used to execute the specific behavior for any value or pointer of a given type that implements the interface.  
  
Let’s implement the Notify method for our User type and call the SendNotification function passing a value of type User:  
  
```go
func (u *User) Notify() error {  
    log.Printf("User: Sending User Email To %s<%s>\n",  
        u.Name,  
        u.Email)  
  
    return nil  
}  
  
func main() {  
    user := User{  
        Name:  "janet jones",  
        Email: "janet@email.com",  
    }  
  
    SendNotification(user)  
}  
  
```

// Output:  
cannot use user (type User) as type Notifier in function argument:  
      User does not implement Notifier (Notify method has pointer receiver)

  
[http://play.golang.org/p/VieiPRDGVu](http://play.golang.org/p/VieiPRDGVu)  
  
Why does the compiler not consider our value to be of a type that implements the interface? The rules for determining interface compliance are based on the receiver for those methods and how the interface call is being made. Here are the rules in the spec for how the compiler determines if the value or pointer for our type [implements](http://golang.org/ref/spec#Method_sets) the interface:  

- _The method set of the corresponding pointer type *T is the set of all methods with receiver *T or T_

This rule is stating that if the interface variable we are using to call a particular interface method contains a pointer, then methods with receivers based on both values and pointers will satisfy the interface. This rule does not apply for our example because we are passing a value to the SendNotification function.  

- _The method set of any other type T consists of all methods with receiver type T._

This rule is stating that if the interface variable we are using to call a particular interface method contains a value, then only methods with receivers based on values will satisfy the interface. This rule does not apply for our example because the receiver for the Notify method accepts a pointer.  
  
Since those are the only two rules in the spec for interface compliance, I have derived this rule that applies to our example:  

- _The method set of the corresponding type T **does not** consists of any methods with receiver type *T._

This is our case and why we are receiving the compiler error. The Notify method is using a pointer for the receiver and we are using a value to make the interface method call. To fix this we just need to pass the address of the User value to the SendNotification function:  
  
```go
func main() {  
    user := &User{  
        Name:  "janet jones",  
        Email: "janet@email.com",  
    }  
  
    SendNotification(user)  
}  
```

  
// Output:  
User: Sending User Email To janet jones<janet@email.com>

  
[http://play.golang.org/p/3NNiS4dMrK](http://play.golang.org/p/3NNiS4dMrK)  
  
**Embedded Types**  
[Struct types](http://golang.org/ref/spec#Struct_types) have the ability to contain anonymous or embedded fields. This is also called embedding a type. When we embed a type into a struct, the name of the type acts as the field name for what is then an embedded field.  
  
Let’s declare a new type and embed our User type into it:  
  
```go
type Admin struct {  
    User  
    Level string  
}
```


  
We have declared a new type called Admin and embedded the User type within the struct declaration. This is not inheritance but composition. There is no relationship between the User and the Admin type.  
  
Let’s change main to create a value of the Admin type and pass the address of this value to the SendNotification function:  
  
```go
func main() {  
    admin := &Admin{  
        User: User{  
            Name:  "john smith",  
            Email: "john@email.com",  
        },  
        Level: "super",  
    }  
  
    SendNotification(admin)  
}  
```

  
// Output  
User: Sending User Email To john smith<john@email.com>

  
[http://play.golang.org/p/2jZMCGEfxW](http://play.golang.org/p/2jZMCGEfxW)  
  
Sure enough, we are able to call the SendNotification function with a pointer of type Admin. Thanks to composition, the Admin type now implements the interface through the promotion of the methods from the embedded User type.  
  
If the Admin type now contains the fields and methods of the User type, then where are they in relationship to the struct?  
  
_"When we [embed](http://golang.org/doc/effective_go.html#embedding) a type, the methods of that type become methods of the outer type, but when they are invoked, the receiver of the method is the inner type, not the outer one." - Effective Go_  
  
Since the name of the embedded type acts as the field name and the embedded type exists as an inner type, we can then make the following method call:  
  
```go
admin.User.Notify()  

```
  
// Output  
User: Sending User Email To john smith<john@email.com>

  
[http://play.golang.org/p/_huNeKVmXS](http://play.golang.org/p/_huNeKVmXS)  
  
Here we are accessing the field and method set of the inner type through the use of the type’s name. However, these fields and methods are also promoted to the outer type:  
  
```go
admin.Notify()  
```
  
// Output  
User: Sending User Email To john smith<john@email.com>

  
[http://play.golang.org/p/v4ro-KHiKJ](http://play.golang.org/p/v4ro-KHiKJ)  
  
So calling the Notify method using the outer type, calls the implementation of the inner type’s method.  
  
These are the rules for inner type [method set promotion](http://golang.org/ref/spec#Method_sets) in Go:  
  
_Given a struct type S and a type named T, promoted methods are included in the method set of the struct as follows:_  

- _If S contains an anonymous field T, the method sets of S and *S both include promoted methods with receiver T._

This rule is stating that when we embed a type, methods for the embedded type with receivers that use a value are promoted and available for calling by values and pointers of the outer type.  

- _The method set of *S also includes promoted methods with receiver *T._

This rule is stating that when we embed a type, methods for the embedded type with receivers that use a pointer are promoted and available for calling by pointers of the outer type.  

- _If S contains an anonymous field *T, the method sets of S and *S both include promoted methods with receiver T or *T._

This rule is stating that when we embed a pointer of the type, methods for the embedded type with receivers that use both values and pointers are promoted and available for calling by values and pointers of the outer type.  
  
Since those are the only three rules in the spec for method promotion, I have derived this rule for one other case:  

- _If S contains an anonymous field T, the method set of S **does not** include promoted methods with receiver *T._

This rule is stating that when we embed a type, methods for the embedded type with receivers that use a pointer are not promoted for calling by values of the outer type. This is consistent with the rules for interface compliance that we stated above.  
  
**Answering The Questions**  
Now we can finalize the sample program that will provide the answers for the two questions we asked in the beginning of the post. Let’s implement the Notifier interface for the Admin type:  
  
```go
func (a *Admin) Notify() error {  
    log.Printf("Admin: Sending Admin Email To %s<%s>\n",  
        a.Name,  
        a.Email)  
  
    return nil  
}
```


  
The implementation of the interface by the Admin type displays a message on behalf of an admin. This will help us determine which implementation gets called when we use a pointer of the Admin type to make the function call to SendNotification.  
  
Now let’s create a value of the Admin type and pass the address of that value to the SendNotification function and see what happens:  
  
```go
func main() {  
    admin := &Admin{  
        User: User{  
            Name:  "john smith",  
            Email: "john@email.com",  
        },  
        Level: "super",  
    }  
  
    SendNotification(admin)  
}  
```

  
// Output  
Admin: Sending Admin Email To john smith<john@email.com>

  
[http://play.golang.org/p/NkDioPJs04](http://play.golang.org/p/NkDioPJs04)  
  
As expected, the Admin type’s implementation of the interface is called by the SendNotification function. So now what happens when we call the Notify method using the outer type:  
  
```go
admin.Notify()  
```
  
// Output  
Admin: Sending Admin Email To john smith<john@email.com>

  
[http://play.golang.org/p/RG50rxC0d7](http://play.golang.org/p/RG50rxC0d7)  
  
We get the output for the Admin type’s implementation. The User type’s implementation is no longer promoted to the outer type:  
  
So now we have the knowledge we need to answer the questions:  

- Would the compiler throw an error because we now had two implementations of the interface?

No, because when we use an embedded type, the unqualified type’s name acts as the field name. This has the effect of fields and methods of the embedded type having a unique name as an inner type of the struct. So we can have an inner and outer implementation of the same interface with each implementation being unique and accessible.  

- If the compiler accepted the type declaration, how does the compiler determine which implementation to use for interface calls?

If the outer type contains an implementation that satisfies the interface, it will be used. Otherwise, thanks to method promotion, any inner type that implements the interface can be used through the outer type.  
  
**Conclusion**  
The way methods, interfaces, and embedded types work together is something that makes Go very unique. These features help us create powerful constructs to achieve the same ends as object oriented code without all the complexity. With the language features that we talked about in this post, we can build abstracted and scalable frameworks with a minimal amount of code and confusion.  

  
The more I learn about the details of the language and the compiler, the more I come to appreciate how [orthogonal](http://en.wikipedia.org/wiki/Orthogonality_\(programming\)) the language is. Small features that work together and allow us to be creative and use the language in ways not even the language designers thought or dreamed about. I recommend to take the time to learn the language features so you can do more with less and be both creative and productive at the same time.