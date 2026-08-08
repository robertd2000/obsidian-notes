https://alexanderobregon.substack.com/p/goroutine-stacks-in-go

[

![](https://substackcdn.com/image/fetch/$s_!RDLG!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa067f2b1-6891-4c5a-8661-ac0b537751e4_756x283.png)



](https://substackcdn.com/image/fetch/$s_!RDLG!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa067f2b1-6891-4c5a-8661-ac0b537751e4_756x283.png)

[Image Source](https://go.dev/)

Concurrency in Go runs on goroutines, which are far lighter than operating system threads. What makes this possible is how their stacks are handled. Instead of reserving a large stack up front like a thread, a goroutine begins with a very small one that grows as the workload demands more space. The Go runtime expands it automatically, moving data and fixing up pointers as needed. This lets millions of goroutines share the same process without draining memory, while still giving them the depth to handle long call chains safely.

Subscribe

### How Goroutine Stacks Start Out Small

Goroutines stand apart from operating system threads because of how their stacks are managed from the very beginning. Each one is given a tiny starting stack, far smaller than what a native thread would expect. This design keeps memory overhead low, yet it still allows the runtime to grow the stack when needed. The starting conditions set the stage for how efficiently Go can juggle thousands or even millions of concurrent executions without wasting resources.

#### Initial Allocation

Goroutines start with a small stack. The minimum is 2 KB stack sizes are powers of two, and the runtime picks an initial size adaptively based on recent stack usage. Some platforms add a small per-OS allowance. Compare that to an OS thread, which often reserves a full megabyte or more at creation. The small footprint is deliberate because many goroutines don’t need much stack space, so starting small avoids overcommitting memory. If more depth is required, the runtime grows the stack gradually rather than paying the cost up front.

You can see this small starting stack in action by writing a function that doesn’t do much beyond a couple of calls. The runtime won’t need to expand the stack because the frames remain shallow.

[

![](https://substackcdn.com/image/fetch/$s_!q6yi!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7c361c41-b868-4a6b-8e5e-4b4199968e6b_1185x503.png)



](https://substackcdn.com/image/fetch/$s_!q6yi!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7c361c41-b868-4a6b-8e5e-4b4199968e6b_1185x503.png)

A goroutine running `simple` doesn’t touch more than a few hundred bytes of its 2 KB allocation. No extra memory needs to be requested, and it happily completes with the starter stack.

On the other hand, recursion or heavy function chains force the runtime to expand beyond that 2 KB. The important detail here is that Go never commits large stacks for every goroutine by default, making it feasible to run vast numbers of them side by side.

#### Stack Layout

The stack for a goroutine is a contiguous region of memory. Each function call places a frame on that stack, and the frame holds arguments, return addresses, and local variables. The compiler decides exactly how the frame is laid out, but the general rule is that newer calls push deeper down. When a function returns, its frame is popped, reclaiming the space.

A simple loop that calls a function repeatedly shows how frames appear and disappear without needing much stack space overall:

[

![](https://substackcdn.com/image/fetch/$s_!d5e0!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff3ac94fc-8bb3-415a-8ed8-1f5586d2b80c_1189x546.png)



](https://substackcdn.com/image/fetch/$s_!d5e0!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff3ac94fc-8bb3-415a-8ed8-1f5586d2b80c_1189x546.png)

Even though the function `printValue` is called many times, each call frame vanishes when the function exits. The goroutine never has more than a couple of active frames at once, which keeps stack usage tiny.

Things change if a function allocates larger local variables. An array declared inside a frame consumes stack space directly, and repeating that through nested calls quickly increases demand.

[

![](https://substackcdn.com/image/fetch/$s_!ipwZ!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6c04346f-6d45-4ed3-a726-2aa4cc42dc5e_1193x296.png)



](https://substackcdn.com/image/fetch/$s_!ipwZ!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6c04346f-6d45-4ed3-a726-2aa4cc42dc5e_1193x296.png)

A recursive call chain through `heavyFrame` builds up several half-kilobyte allocations on the stack. That’s where the layout matters, because each nested call consumes additional space until the runtime needs to expand the stack.

#### Stack Guards

Go uses a guard system to keep goroutines from silently running past their stack boundary. Each goroutine stores a guard value in its internal control structure. Each function prologue compares the stack pointer against the goroutine’s `stackguard` value. If the frame would violate the guard, it calls `morestack` to grow the stack before continuing. If the call would leave too little space, the runtime intervenes by growing the stack.

You can trigger this behavior by writing a recursive function with a deeper call count. The stack guard fires when the runtime predicts that the next call would overflow the current space.

[

![](https://substackcdn.com/image/fetch/$s_!Sm-p!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe138307f-cf91-4e76-95bf-7f399a316bc4_1232x420.png)



](https://substackcdn.com/image/fetch/$s_!Sm-p!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe138307f-cf91-4e76-95bf-7f399a316bc4_1232x420.png)

What’s happening here is that the runtime compares the current stack pointer against the guard threshold before each new call. After the threshold is crossed, stack growth is requested instead of letting the goroutine wander into memory it doesn’t own.

Another way the guard helps is during large local allocations. If a function frame needs more than what’s left between the pointer and the guard, it can’t proceed until the stack is expanded. This check runs automatically, which means developers don’t have to manually keep track of stack limits.

### How Goroutine Stacks Grow

Small stacks are only the beginning. When a goroutine needs more room to handle deeper calls or larger frames, the Go runtime expands its stack automatically. This process is invisible to the developer but involves a surprising amount of behind-the-scenes work. Expansion is not done in tiny increments but in large steps that balance memory efficiency with performance.

#### Doubling Strategy

Stack sizes are powers of two. When growth is needed, the runtime copies the current stack to a larger one, rounding up to the next power of two that fits the frame that triggered growth which in practice that often means doubling. The contents of the old stack are copied into the new space, and execution continues from where it left off. Doubling helps keep the number of reallocations small, which matters because copying stacks takes time.

On 64-bit systems the default limit for a single goroutine stack is `~1 GB` and `~250 MB` on 32-bit, configurable via `runtime/debug.SetMaxStack`. Most stacks never get anywhere near that size because practical workloads rarely need such depth, but the upper limit is there for extreme cases.

You can create a stress test that quickly pushes a stack beyond its initial allocation with recursion:

[

![](https://substackcdn.com/image/fetch/$s_!awfk!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0f64093c-57a6-4fdc-bb71-6c326b9c0f68_1167x588.png)



](https://substackcdn.com/image/fetch/$s_!awfk!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0f64093c-57a6-4fdc-bb71-6c326b9c0f68_1167x588.png)

Each call in this example consumes an extra kilobyte, forcing the runtime to grow the stack repeatedly. The doubling strategy means the new stack size progresses from 2 KB to 4 KB, then 8 KB, then 16 KB, and so on, until the call depth fits.

#### Pointer Adjustment

Growing a stack is not as simple as copying bytes. Pointers that referenced variables on the old stack must be corrected, or they’d end up pointing to invalid memory. The runtime works with the garbage collector to locate those references and update them to the new addresses. This process happens every time a stack expands.

Go doesn’t keep outside pointers into another goroutine’s stack. When the address of a local variable would escape, the compiler places it on the heap instead of leaving it on the stack. That’s why a snippet like `saved = append(saved, &x)` doesn’t actually show stack growth. The variable `x` is moved to the heap, and what you get back is a safe heap pointer. When a stack really does expand, the runtime copies its contents into a larger block and fixes every pointer inside that stack so execution continues without breaking references. Runtime structures that track goroutines, like channel waiters, are also updated so they point to the new location. Outside code never gets a pointer into a goroutine’s stack, so nothing external needs fixing.

You can see whether the compiler decided to move a local off the stack by asking it to print escape analysis results:

[

![](https://substackcdn.com/image/fetch/$s_!jQxu!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fff0bab92-6b51-47f2-b0cb-0d5e62f717a9_1196x37.png)



](https://substackcdn.com/image/fetch/$s_!jQxu!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fff0bab92-6b51-47f2-b0cb-0d5e62f717a9_1196x37.png)

That output will show when a variable has been promoted to the heap, which is the mechanism that keeps pointers safe across stack moves rather than the runtime trying to preserve raw references.

#### Shrinking Stacks

Growth is only one side of the story. A goroutine may not always need a large stack, especially after finishing a period of heavy recursion. Stacks shrink during garbage collection. After scanning a goroutine’s stack, the runtime may move it to a smaller power-of-two size if little of it is live. The decision and the copy happen in GC code paths.

Shrinking doesn’t happen constantly because resizing too often would waste resources. Instead, the runtime piggybacks on the garbage collector’s scans, which already walk through goroutine stacks to mark live data. If a large stack has a small active section, the collector can release the unused portion.

You can write code that grows a stack temporarily, then falls back to light calls, which often leads to shrinking later on:

[

![](https://substackcdn.com/image/fetch/$s_!Uzlt!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1d6a730b-0fca-4d1a-818b-5c03a33751c5_1160x713.png)



](https://substackcdn.com/image/fetch/$s_!Uzlt!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1d6a730b-0fca-4d1a-818b-5c03a33751c5_1160x713.png)

When garbage collection runs during the loop, the runtime may notice that only a small portion of the stack is active and shrink it back down. This prevents long-lived goroutines from holding on to more memory than they need.

#### Growth with Shrink Example

A more complete picture comes from code that combines periods of deep recursion with phases of shallow processing. This simulates real workloads where a goroutine alternates between heavy call chains and light loops.

[

![](https://substackcdn.com/image/fetch/$s_!aH36!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F501ee114-d921-4263-b795-a925d1b89244_1024x768.png)



](https://substackcdn.com/image/fetch/$s_!aH36!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F501ee114-d921-4263-b795-a925d1b89244_1024x768.png)

The recursion drives stack growth through repeated allocation of 256-byte buffers. After the recursion unwinds, the loop phase runs with shallow frames. During later garbage collection passes, the runtime may shrink the stack, reducing memory usage for the goroutine while it continues running.

The code here shows both sides of stack management. Growth makes sure that the goroutine has the space to complete complex call chains, and shrinkage allows it to release that space when it’s no longer needed.

### Conclusion

Goroutine stacks are designed to start tiny, grow in measured steps, and return memory when it’s no longer required. The runtime handles allocation, copying, and pointer correction automatically, letting stacks expand for deep calls and shrink during lighter phases. That balance is what makes it possible for Go to support massive numbers of goroutines without wasting space, while still keeping execution safe when call chains run deeper than expected.

1. _[Goroutines — Go Documentation](https://go.dev/doc/effective_go#goroutines)_
    
2. _[Stack Growth — Go Runtime Source Notes](https://github.com/golang/go/blob/master/src/runtime/stack.go)_
    
3. _[The Go Memory Model](https://go.dev/ref/mem)_
    
4. _[Go Garbage Collector Design](https://go.dev/doc/gc-guide)_