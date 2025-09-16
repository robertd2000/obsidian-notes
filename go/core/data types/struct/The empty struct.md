https://dave.cheney.net/2014/03/25/the-empty-struct

# Introduction

This post explores the properties of my favourite Go data type, the _empty struct_.

The empty struct is a [struct type](http://golang.org/ref/spec#Struct_types) that has no fields. Here are a few examples in named and anonymous forms

type Q struct{}
var q struct{}

So, if an empty struct contains no fields, contains no data, what use is it ? What can we do with it ?

# Width

Before diving into the empty struct itself, I wanted to take a brief detour to talk about _width_.

The term width comes, like most terms, from the gc compiler, although its etymology probably goes back decades.

Width describes the number of bytes of storage an instance of a type occupies. As a process’s address space is one dimensional, I think width is a more apt than size.

Width is a property of a type. As every value in a Go program has a type, the width of the value is defined by its type and is always a multiple of 8 bits.

We can discover the width of any value, and thus the width of its type using the `unsafe.Sizeof()` function.

var s string
var c complex128
fmt.Println(unsafe.Sizeof(s))	 // prints 8
fmt.Println(unsafe.Sizeof(c))	 // prints 16

[http://play.golang.org/p/4mzdOKW6uQ](http://play.golang.org/p/4mzdOKW6uQ)

The width of an array type is a multiple of its element type.

var a [3]uint32
fmt.Println(unsafe.Sizeof(a)) // prints 12

[http://play.golang.org/p/YC97xsGG73](http://play.golang.org/p/YC97xsGG73)

Structs provide a more flexible way of defining composite types, whose width is the sum of the width of the constituent types, plus padding

type S struct {
        a uint16
        b uint32
}
var s S
fmt.Println(unsafe.Sizeof(s)) // prints 8, not 6

~~The example above demonstrates one aspect of padding, that a value must be aligned in memory to a multiple of its width. In this case there are two bytes of padding added by the compiler between `a` and `b`.~~

**Update** Russ Cox has kindly written to explain that width is unrelated to alignment. You can [read his comment below](http://dave.cheney.net/2014/03/25/the-empty-struct#comment-2815).

# An empty struct

Now that we’ve explored width it should be evident that the empty struct has a width of zero. It occupies zero bytes of storage.

var s struct{}
fmt.Println(unsafe.Sizeof(s)) // prints 0

As the empty struct consumes zero bytes, it follows that it needs no padding. Thus a `struct` comprised of empty structs also consumes no storage.

type S struct {
        A struct{}
        B struct{}
}
var s S
fmt.Println(unsafe.Sizeof(s)) // prints 0

[http://play.golang.org/p/PyGYFmPmMt](http://play.golang.org/p/PyGYFmPmMt)

# What can you do with an empty struct

True to Go’s orthogonality, an empty struct is a `struct` type like any other. All the properties you are used to with normal structs apply equally to the empty struct.

You can declare an array of `structs{}`s, but they of course consume no storage.

var x [1000000000]struct{}
fmt.Println(unsafe.Sizeof(x)) // prints 0

[http://play.golang.org/p/0lWjhSQmkc](http://play.golang.org/p/0lWjhSQmkc)

Slices of `struct{}`s consume only the space for their slice header. As demonstrated above, their backing array consumes no space.

var x = make([]struct{}, 1000000000)
fmt.Println(unsafe.Sizeof(x)) // prints 12 in the playground

[http://play.golang.org/p/vBKP8VQpd8](http://play.golang.org/p/vBKP8VQpd8)

Of course the normal subslice, `len`, and `cap` builtins work as expected.

var x = make([]struct{}, 100)
var y = x[:50]
fmt.Println(len(y), cap(y)) // prints 50 100

[http://play.golang.org/p/8cO4SbrWVP](http://play.golang.org/p/8cO4SbrWVP)

You can take the address of a `struct{}` value, when it is [addressable](http://golang.org/ref/spec#Address_operators), just like any other value.

var a struct{}
var b = &a

Interestingly, the address of two `struct{}` values may be the same.

var a, b struct{}
fmt.Println(&a == &b) // true

[http://play.golang.org/p/uMjQpOOkX1](http://play.golang.org/p/uMjQpOOkX1)

This property is also observable for `[]struct{}`s.

a := make([]struct{}, 10)
b := make([]struct{}, 20)
fmt.Println(&a == &b)       // false, a and b are different slices
fmt.Println(&a[0] == &b[0]) // true, their backing arrays are the same

[http://play.golang.org/p/oehdExdd96](http://play.golang.org/p/oehdExdd96)

Why is this? Well if you think about it, empty structs contain no fields, so can hold no data. If empty structs hold no data, it is not possible to determine if two `struct{}` values are different. They are in effect, fungible.

a := struct{}{} // not the zero value, a real new struct{} instance
b := struct{}{}
fmt.Println(a == b) // true

[http://play.golang.org/p/K9qjnPiwM8](http://play.golang.org/p/K9qjnPiwM8)

_note:_ this property is not required by the spec, but it does note that [Two distinct zero-size variables may have the same address in memory.](http://golang.org/ref/spec#Size_and_alignment_guarantees)

# struct{} as a method receiver

Now we’ve demonstrated that empty structs behave just like any other type, it follows that we may use them as method receivers.

type S struct{}

func (s *S) addr() { fmt.Printf("%p\n", s) }

func main() {
        var a, b S
        a.addr() // 0x1beeb0
        b.addr() // 0x1beeb0
}

[http://play.golang.org/p/YSQCczP-Pt](http://play.golang.org/p/YSQCczP-Pt)

In this example it shows that the address of _all_ zero sized values is 0x1beeb0. The exact address will probably vary for different versions of Go.

# Wrapping up

Thank you for reading this far. At close to 800 words this article turned out to be longer than expected, and there was still more I was planning to write.

While this article concentrated on language obscura, there is one important practical use of empty structs, and that is the `chan struct{}` construct used for signaling between go routines

I’ve talked about the use of `chan struct{}` in my [Curious Channels](http://dave.cheney.net/2013/04/30/curious-channels "Curious Channels") article.