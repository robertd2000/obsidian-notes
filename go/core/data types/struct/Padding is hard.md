https://dave.cheney.net/2015/10/09/padding-is-hard

We all know that [the empty struct consumes no storage](http://dave.cheney.net/2014/03/25/the-empty-struct), right ? Here is a curious case where this turns out to not be true.

[![Memory Profile](https://dave.cheney.net/wp-content/uploads/2015/10/Screenshot-from-2015-10-09-153350.png)](https://dave.cheney.net/wp-content/uploads/2015/10/Screenshot-from-2015-10-09-153350.png)

This shows the “after” graph, prior to [CL 15522](https://golang.org/cl/15522) each allocation was 320 bytes.

This is a story about trying to speed up the Go compiler. Since Go 1.5 we’ve had the great concurrent GC, which reduces the cost of garbage collection, but no matter how efficient a garbage collector is, it doesn’t make allocations free of cost.

The specific allocation I was targeting was the `obj.Prog` structure which represents a quasi machine code instruction late in the compile cycle. As you can see, for this compilation (which was `cmd/compile/internal/gc`) `obj.Prog` values constitute 34.72% of the total allocations for this program—reducing the size of `obj.Prog` will pay off in real terms.

[obj.Prog](https://github.com/golang/go/blob/master/src/cmd/internal/obj/link.go#L202) itself contains a field of type [obj.ProgInfo](https://github.com/golang/go/blob/master/src/cmd/internal/obj/link.go#L227), which is what this blog post will focus on today. Here is the definition for `obj.ProgInfo`

```go
type ProgInfo struct {
        Flags    uint32   // flag bits
        Reguse   uint64   // registers implicitly used by this instruction
        Regset   uint64   // registers implicitly set by this instruction
        Regindex uint64   // registers used by addressing mode
        _        struct{} // to prevent unkeyed literals
}
```


How much space will values of this type consume ? TL;DR, here is a table of the results:

![[Pasted image 20250915105912.png]]

Go 1.4 32 bit says the value is 28 bytes large, but Go 1.4 64 bit says it’s 32 bytes large. Worse, Go 1.5 32 bit says the value is 32 bytes, and Go 1.5 64 bit says it’s 40 bytes! This type uses fixed sized integer types, but its total size changes every time we compile it. What the heck is going on here ?

[![Mind=Blown](https://dave.cheney.net/wp-content/uploads/2015/10/STDryl1.gif)](https://dave.cheney.net/wp-content/uploads/2015/10/STDryl1.gif)

## Alignment and padding

The reason for the difference between the 32 bit and 64 bit answers is alignment. The Go spec says the address of a struct’s fields must be naturally aligned. So on a 64 bit machine, the address of any `uint64` field must be a multiple of 8 bytes. In effect the compiler sees this:

```go

type ProgInfo struct {
        Flags    uint32   // flag bits
        **_        [4]byte  // padding added by compiler**
        Reguse   uint64   // registers implicitly used by this instruction
        Regset   uint64   // registers implicitly set by this instruction
        Regindex uint64   // registers used by addressing mode
        _        struct{} // to prevent unkeyed literals
}


```

On 32 bit machines, word boundaries occur every 4 bytes, so there is no requirement to _pad_ for alignment. Note that operations on 64 bit quantities may still require the value to be properly aligned, this is the infamous [issue 599](https://golang.org/issue/599).

So, that explains the difference in results between 32 bit and 64 bit machines, `Flags` (4 bytes) + `Reguse` (8 bytes) + `Regset` (8 bytes) + `Regindex` (8 bytes) == 28 bytes on 32 bit machines, plus another 4 bytes for padding for 64 bit machines.

Well, except that Go 1.5 seems to be consuming another word over and above the Go 1.4 requirements, where is this memory going ?

To give you a hint, I’ll rearrange the definition slightly:

```go
type ProgInfo struct {
        Reguse   uint64   // registers implicitly used by this instruction
        Regset   uint64   // registers implicitly set by this instruction
        Regindex uint64   // registers used by addressing mode
        **Flags    uint32   // flag bits**
        _        struct{} // to prevent unkeyed literals
}
```


Now the results are:

![[Pasted image 20250915105937.png]]

Hmm, so some progress. We’ve managed to reduce the storage in the Go 1.5 64 bit case, but the others appear unchanged, especially Go 1.5’s 32 bit case.

For the Go 1.5 64 bit case, the padding that we saw above is still there, but it has moved. Effectively what the compiler is seeing is this:

```go

type ProgInfo struct {
        Reguse   uint64   // registers implicitly used by this instruction
        Regset   uint64   // registers implicitly set by this instruction
        Regindex uint64   // registers used by addressing mode
        Flags    uint32   // flag bits
        **_        [4]byte  // padding for alignment of ProgInfo**
        _        struct{} // to prevent unkeyed literals
}

```


The reason this padding is still present is to preserve the alignment of the other fields in this structure. Remember that `uint64` fields always have to be naturally aligned, every 4 bytes on 32 bit platforms, and every 8 bytes on 64 bit platforms. Consider this small fragment:

var pp [4]ProgInfo	
fmt.Println(&pp[1].Reguse) // **address must be a multiple of 8 on 64 bit platforms**

The compiler adds padding at the end of the structure to bring its size up to a [multiple of the largest element in the structure](https://golang.org/ref/spec#Size_and_alignment_guarantees).

So, that explains everything except for the Go 1.5 32 bit result. For 32 bit platforms, the structure needs to be aligned on a 4 byte boundary, not 8, so no padding should be required. The size under both Go 1.4 and Go 1.5 should be 28 bytes, not 32 bytes … what is going on ?

## That empty struct

You’ve probably figured out by now that the only place the additional storage could come from is the unnamed `struct{}` field. What is it about the presence of an empty struct at the bottom of the type that causes it to increase the size of the struct ?

The answer is that while empty struct{} values consume no storage, you can _take their address_. That is, if you have a type

```go
type T struct {
      X uint32
      Y struct{}
}

var t T
```


It is perfectly valid to take the address of `t.Y`, and as such, the address of `t.Y` would point _beyond the end of the struct!_1

Now, Go code cannot do anything useful with this invalid pointer, but as it is a pointer, the garbage collector has to follow it, and that may lead it to access unmapped memory or dereference an invalid pointer.

The fix for this is detailed in [issue 9401](https://github.com/golang/go/issues/9401) and delivered in Go 1.5. In summary, zero sized values, like `struct{}` and `[0]byte` occurring at the end of a structure are assumed to have a size of one byte2. With this knowledge, we can now explain the behaviour of the compiler for the rearranged type.

```go
type ProgInfo struct {
        Reguse   uint64   // registers implicitly used by this instruction
        Regset   uint64   // registers implicitly set by this instruction
        Regindex uint64   // registers used by addressing mode
        Flags    uint32   // flag bits
        _        struct{} // to prevent unkeyed literals
        **_        [1]byte  // to prevent unkeyed literals**  
}
```


For a 64 bit compiler, it would observe that the largest field inside `ProgInfo` is 8 bytes, so it would add three additional bytes of padding after the `[1]byte` to round up to 32 bytes. This is fine, it was already planning to add 4 bytes after `Flags`.

For a 32 bit compiler, it would observe the largest field inside `ProgInfo` is 8 bytes, however the natural alignment of 32 bit machines is 4 bytes and the sum of the fields inside the type is 28 bytes, which is what Go 1.4 reported. However because the final field is of zero length, the compiler has replaced it with a `[1]byte`, causing the total size of the structure to be 29 bytes, forcing the compiler to add three more bytes of padding to round up to the next word boundary, 32 bytes.

Fortunately this last issue can be corrected by [moving the zero field to the top of the structure](https://go-review.googlesource.com/#/c/15660/), restoring the original sizes we observed in Go 1.4.

## Wrapping up

Do Go programmers need to care about this minutiae ? Probably not. This is the stuff of brain teasers.

Certainly if you are looking to optimise the memory usage of your program, you need to be looking closely at the definitions of all your data structures, but it is unlikely that you’ll encounter the perfect storm of all these factors.

Not to be outdone by this morning’s excitement, Matthew Dempsky put together a quick tool to [spot this inefficiency](https://github.com/mdempsky/maligned), and has [found two other candidates](https://github.com/golang/go/issues/12884#issuecomment-146758684).

## One more thing

At the start of this piece I mentioned that [CL 15522](https://go-review.googlesource.com/#/c/15522/) resulted in a saving of 32 bytes, yet the biggest saving demonstrated above was 8 bytes. How did CL 15522 achieve such a reduction ? The answer is a thing known inside the runtime as _size classes_.

For small allocations, that is, less than a few kilobytes, the overhead of tracking the allocation becomes a significant percentage of the object itself. Imagine if every allocation had one word of overhead, every time you put an `int` into an `interface`3 value, you’d incur 100% overhead!

To reduce this overhead, rather than tracking every object on the heap individually, the runtime allocates things of the _same size together_. This leads to lower overhead—one bit per object of a certain size, and excellent space efficiency—all objects are of the same size, eliminating fragmentation.

However, as Go types can be any size, potentially the runtime would have to allocate a pool for every possible size (obviously not until they are actually allocated), and this would undo the efficiency of the mechanism as each pool is a number of machine pages, yet most pools would be largely unused.

The solution to this problem is to not create pools for every possible size, but to start to round up the allocation to the next largest _size class_, for example an allocation of 40 bytes may be rounded up to 48 bytes. The details of this mechanism are left to the curiosity of the reader4, but suffice to say, as allocations get larger, the distance between size classes increases.

Hence, by shaving 8 bytes from `obj.ProgInfo`, the size of the enclosing `obj.Prog` type decreased from 296 bytes to 288 bytes, bringing it down to from the previous 320 byte size class, to the 288 byte size class, a saving of 32 bytes on 64 bit platforms.

---

1. The explanation for why this is not a violation of Go’s memory safety guarantee is left as an exercise for the reader.
2. The explanation for why this does not break the semantics of Go programs is left as an exercise for the reader.
3. The explanation for why storing an `int` into an `interface` causes a heap allocation is left as an exercise for the reader.
4. [`runtime/msize.go`](https://github.com/golang/go/blob/master/src/runtime/msize.go)

### Related posts:

1. [The empty struct](https://dave.cheney.net/2014/03/25/the-empty-struct "The empty struct")
2. [Friday pop quiz: the smallest buffer](https://dave.cheney.net/2015/06/05/friday-pop-quiz-the-smallest-buffer "Friday pop quiz: the smallest buffer")
3. [Struct composition with Go](https://dave.cheney.net/2015/05/22/struct-composition-with-go "Struct composition with Go")
4. [Simple extended attribute support for Go](https://dave.cheney.net/2011/07/31/simple-extended-attribute-support-for-go "Simple extended attribute support for Go")