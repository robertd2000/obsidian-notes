https://dave.cheney.net/2013/06/02/why-is-a-goroutines-stack-infinite

Occasionally new Gophers stumble across a curious property of the [Go language](http://golang.org/) related to the amount of stack available to a Goroutine. This typically arises due to the programmer inadvertently creating an infinitely recursive function call. To illustrate this, consider the following (slightly contrived) example.

```go
package main

import "fmt"

type S struct {
        a, b int
}

// String implements the fmt.Stringer interface
func (s *S) String() string {
        return fmt.Sprintf("%s", s) // Sprintf will call s.String()
}

func main() {
        s := &S{a: 1, b: 2}
        fmt.Println(s)
}
```


Were you to run this program, and I do not suggest that you do, you’d find that your machine would start to swap heavily, and will probably become unresponsive unless you’re quick to hit ^C before things become unsalvageable  Because I know the first thing everyone will do is try to run this program in the playground, [I’ve saved you the bother](http://play.golang.org/p/XzZsmGLgIp).

Most programmers have run into problems with infinite recursion before, and while it is fatal to their program, it isn’t usually fatal to their machine. So, why are Go programs different ?

One of the key features of Goroutines is their cost; they are cheap to create in terms of initial memory footprint (as opposed to the 1 to 8 megabytes with a traditional POSIX thread) and their stack grows and shrinks as necessary. This allows a Goroutine to start with a single 4096 byte stack which grows and shrinks as needed without the risk of ever running out.

To implement this the linker (`5l`, `6l`, `8l`) inserts a small preamble at the start of each function1, which checks to see if the amount of stack required for the function is below the amount currently available. If not, a call is made to `runtime⋅morestack,` which allocates a new stack page2, copies the arguments from the caller, then returns control to the original function which can now execute safely. When that function exits, the process is undone, its return arguments are copied back to the stack frame of the caller and the unneeded stack space released.

By this process the stack is effectively infinite, and assuming that you’re not continually straddling the boundary between two stacks, colloquially known as _stack splitting_, is very cheap.

There is however one detail I have withheld until now, which links the accidental use of a recursive function to a serious case of memory exhaustion for your operating system, and that is, when new stack pages are needed, they are allocated _from the heap_.

As your infinite function continues to call itself, new stack pages are allocated from the heap, permitting the function to continue to call itself over and over again. Fairly quickly the size of the heap will exceed the amount of free physical memory in your machine, at which point swapping will soon make your machine unusable.

The size of the heap available to Go programs depends on a lot of things, including the architecture of your CPU and your operating system, but it generally represents an amount of memory that exceeds the physical memory of your machine, so your machine is likely to swap heavily before your program ever exhausts its heap.

In Go 1.1 there was a strong desire to increase the maximum size of the heap for both 32 bit and 64 bit platforms, and this has exacerbated the problem to some extent, ie, it is unlikely that you will have 128Gb3 of physical memory in your system.

As a final comment, there are several open issues ([link](https://code.google.com/p/go/issues/detail?id=4692), [link](https://code.google.com/p/go/issues/detail?id=2556)) regarding this problem, but a solution that does not extract a performance penalty on properly written programs has yet to be found.

##### Notes

1. This also applies to methods, but as methods are implemented as functions where the first argument is the method receiver, there is no practical difference when discussion how segmented stacks work in Go.
2. Using the word page does not imply that only fixed, 4096 byte, allocations are possible, if necessary `runtime⋅morestack` will allocate a larger amount, probably rounded to a page boundary.
3. 64 bit Windows platforms only permit a 32Gb heap due to a late change in the Go 1.1 release cycle.

### Related posts:

1. [Mid-stack inlining in Go](https://dave.cheney.net/2020/05/02/mid-stack-inlining-in-go "Mid-stack inlining in Go")
2. [Stack traces and the errors package](https://dave.cheney.net/2016/06/12/stack-traces-and-the-errors-package "Stack traces and the errors package")
3. [Never start a goroutine without knowing how it will stop](https://dave.cheney.net/2016/12/22/never-start-a-goroutine-without-knowing-how-it-will-stop "Never start a goroutine without knowing how it will stop")
4. [Inlining optimisations in Go](https://dave.cheney.net/2020/04/25/inlining-optimisations-in-go "Inlining optimisations in Go")