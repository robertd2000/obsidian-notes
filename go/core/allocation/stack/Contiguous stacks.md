Allocate each Go routine a contiguous piece of memory for its stack, grown by reallocation/copy when it fills up.

Why?

- Current split stack mechanism has a “hot split” problem - if the stack is almost full, a call will force a new stack chunk to be allocated.  When that call returns, the new stack chunk is freed.  If the same call happens repeatedly in a tight loop, the overhead of the alloc/free causes significant overhead.
- Stack allocation/deallocation work is never complete with split stacks - every time the stack size passes a threshold in either direction, extra work is required.

With contiguous stacks, we avoid both of these problems.  Stacks are grown to the size required, and no more work is needed from then on (modulo shrinking, see below).

How?

How can you move a stack?  It turns out that our compiler’s escape analysis provides a very important invariant: pointers to data on the stack are only ever passed down in the call tree.  Any other escape (written to global variables, returned to a parent, written to the heap, etc.) prevents that data from being allocated on the stack.  This invariant means that the only pointers to stack data are in the stack itself (but see below for exceptions), which makes them easy to find and update in case we need to copy the stack.

Overflow check

The stack overflow check is very similar to the overflow check for split stacks.  The only major difference is that we no longer need to know what the argument size is, as there is no longer the need to copy the arguments to a new stack segment.  This simplifies the proliferation of the morestack routine family somewhat, and eliminates the need for most of the NOSPLIT directives.

We also don’t even need to know the frame size.  Just double the stack size and try again, if it still doesn’t fit we’ll hit the overflow check again and double the stack size again.  Repeat until it fits.

Copying

When a stack overflow check triggers:

- Allocate a new stack somewhat bigger than the old stack. We should allocate in exponentially increasing sizes (to limit work done copying), and possibly also round up to a multiple of a nice size (so the allocator can be more efficient).  I’m currently planning on using powers of two sizes and just doubling each realloc.
- Copy the old stack to the new stack.  The stack is treated as an array of *byte and each *byte is copied from old to new.  Each *byte is compared with the bounds of the old stack, and if it is in range it is adjusted to point to the same offset from TOS in the new stack.
- Only real pointers can be adjusted - we need to avoid integers that look like pointers.  So we need a data structure that tells us, for each pointer-sized word on the stack, whether it is a pointer or not.  Fortunately, Carl is generating just such a data structure to be used for precise GC.
- Any pointers into the stack from outside need to be adjusted.  Despite the escape analysis, there are a few.  Those include:

- The function object for a deferred call
- The arguments to a deferred call
- The closure parameters of the function object for a deferred call
- The function object that is being passed to the overflowing frame (it gets passed in a register, not on the stack, at least in x86)
- The closure parameters to the function object that is being passed to the overflowing frame

The same rules apply when copying - we need to know whether the data really is a pointer and if so, whether it points into the old stack.

Reflect.call

Reflect.call calls a function given a pointer to an argument area and an argument size.  The old stack-split mechanism forced a new stack chunk so this variable-sized argument area can be allocated at the top of the stack.  The new mechanism needs to allocate these args anywhere on the stack.  The implementation is simpler if all stack frames are fixed size, so to “simulate” a variable-sized frame we define a set of fixed-size-frame functions in powers-of-two sizes which can be used as trampolines.

reflect.call(function, args, argsize)

        if argsize <= 16: jump reflect.call16(function, args, argsize)

        if argsize <= 32: jump reflect.call32(function, args, argsize)

        if argsize <= 64: jump reflect.call64(function, args, argsize)

…

reflect.call16(function, args, argsize)

        sp -= 16

        copy [args, args+argsize] to [sp, sp+argsize]

        call function

        copy [sp, sp+argsize] to [args, args+argsize]  (for return values)

        sp += 16

        return

I propose a maximum reflect.call arg size of 64K.

The stack copier needs to know what is a pointer and what is not for these frames.  This info can be derived by looking at the signature of the called function. (and for vararg calls also the argument size?)

Stack tracing

Stack tracing code simplifies somewhat because it doesn’t need to deal with splits on the stack.

GC

As in stack tracing, GC walk of the stack for a G is simplified.

Defer/panic/recover

The implementations of these mechanisms change slightly, and are simplified in most instances.  The old mechanisms used stack-split locations to mark where panics are happening.

Testing

For testing, all stacks will be allocated somewhere outside the heap.  When a stack is copied, the old stack will be munmap’d so any errant access to the old stack will trap.

CGo

TBD, I don’t know what the issues are yet.

Shrinking

If a Go routine uses lots of stack at the start of its lifetime but then uses much less stack for the rest of its lifetime, then a lot of stack space is left idle.  We need to recover this space somehow.  Ideas:

- At GC time, if a go routine is not using more than X% of its stack, copy it to a smaller stack.  It will regrow it later if it needs it.
- At GC time, if a go routine is not using more than X% of its stack, lower its stack guard to a smaller value.  If it doesn’t hit the guard by the next GC, then copy to a smaller stack.

My current plan is at GC time, if a go routine is using at most ¼ of its stack, free the bottom ½ of the stack.  I’ll be freeing directly if the stack is big enough to allow it, and copying otherwise.

Starting stacks

All Gs by default start with a minimum-sized stack.  When a G finishes, we deallocate its stack, as it may be large and not what is required by the next go routine which happens to start on that G.  (Or maybe we let the shrinking code handle this?)

We could keep some statistics of how much stack each go routine uses (keyed by function address) and pre-allocate that much stack before starting that go routine.  Statistics can be updated each time a go routine finishes.  It won’t be perfect, for example a go routine whose stack use is data-dependent.  But it may be an adequate starting point.

This mechanism could be used in the split stack world as well, if we were willing to record the maximum stack usage during a Go routine.  It would have to be more conservative, however.  In the split-stack world allocating too large an initial stack can’t be undone.

Disadvantages

- More virtual memory pressure / fragmentation.  Especially with large stacks, finding contiguous memory to place the stack becomes more difficult.  It is possible that some hybrid scheme could be better, for example copying up to a size of 1 MB and splitting after that.  Although it would probably help, I don’t think it is worth the additional implementation complexity to retain both mechanisms.
- Pointer-into-stack restrictions.  All pointers into the stack area need to be strictly accounted for.  Currently, our escape analysis ensures that there are no stray pointers into the stack.  There may be situations in the future where we wish to allow pointers into the stack, and this implementation will preclude them.  I’m not really sure what those situations would be, but this limitation is worth mentioning.
- Probably need to do an audit of the runtime C/asm code to make sure nothing tricky with stack pointers is going on.

Experiments so far

I have a prototype [implementation running](https://www.google.com/url?q=https://codereview.appspot.com/9029047/&sa=D&source=editors&ust=1758214187716430&usg=AOvVaw2sq0hbf_QDVR2kwgUQeew7).  It is not complete yet but it can compile simple examples.

Peano (test/peano.go) modified to run up to 11! is about 10% faster.  It is very sensitive to the growth rate, though, so take it with a lot of salt.  (Doubling the stack size each copy is 10% faster.  Growing the stack by 50% each copy is only 2% faster. Growing by 25% is 20% slower.)

“Hot split” testcase ([stacksplit.go](https://www.google.com/url?q=http://play.golang.org/p/YVRi8hzZt1&sa=D&source=editors&ust=1758214187716989&usg=AOvVaw1_4VjMw_mwHOaoeu6HpWBS)).  Compiled with -gcflags -l to disable inlining.

segmented stacks:

 no split: 1.25925147s  
with split: 5.372118558s   <- triggers hot split problem  
both split: 1.293200571s

contiguous stacks:

 no split: 1.261624848s  
with split: 1.262939769s  
both split: 1.29008309s

Benchmark examples

I modified the segmented stack and contiguous stack implementations to use an environment variable to configure their stack segment size / initial stack size.  We can use this feature to test a benchmark’s sensitivity to stack segment size.  (For these experiments, the contiguous stack size is limited to powers of 2.)

encoding/json/BenchmarkEncode: this one has been complained about historically.  See bug 3787.

![](https://lh7-rt.googleusercontent.com/u/0/docsd/ANYlcfA69Ez0qFpuyRugx6c3qLGZWAK_Q3p-Ld5viLmFTL3rcv-mbsioiwr5DWJRa5TEVPQu4v3gBQEwuE7oZDYbadj-0igNqLS0LqkDr29JMHpGPm_NmcEJCApvKpmGm-ESgDQEI9b_B_ruZA0JQPwU9ti8kVCpwMLw_aU04_fBk-ejZd-uiI7ycgiOlK6SdlbZWyVXhrayrvYtagwdTe7yYBdtNLf6zxWNW1w_CajczStDpkapHljLy-K4qDiJjH0Zq--hSBJ47tvJ2uOYR8VmRsGyVmrHVPML-ECyrwYyjVlHjKfkLdebdOfrqZ9wvmb73eoL9LrgBqYzjA8nqp6tw4Hecot-9-hFekbJ5X4QTJAw2-_W-SIUqh22ul5PGDF457Od-78ER_juOk6QgPUy-7Guft53xWvZfH6LfBnRdI7dlV93ZVKWLjYPoVXl9Lj7A-AFuKW7VHlE91oWLQknHF2SU5e0qCzG3hQYcCnxIuIGynjuiCAlG3zZOXzZ0Ar3kmhBZWkBwv4bUcuersd_l9FT8w89H4nAmWtl6y9FUxuAsPiRM7xN8fGvwjCSnaJ08zROb1NcOaU5zpIcSUszzHcjmQgRr_ZhB2DwmRN_piOZ42IwjgRYXsIsRN28sDZNIGIDLIj2N6Wg2t-O22zXcAFATZzhc25XiWgXQ_mbP2Mk0Val2qdkgd3ptO37eEFgIrkhjcThsV3CpHV99ILmrjFs-xAeBWU7rGwLkWDCGLiiK0RPEcEYxQGhEbm2tqvQYGFjDE7jnEYWxaTVxxTqh0etyedQ_mrc3ZgfusXhklJfcseJ20LdKLkUYVfGIEa8dYMgDhPcMMzYJ7JG3k3YD5zf4jddxYmtdiKkH5lnYsTdxelPp3DRxHb9N0XrcXqbY9FnipzToIhZmUM9tzYz)

Presumably the max stack use is 19KB or so, and below that bad splits cause the downward spikes.

html/template/BenchmarkEscapedExecute: this benchmark has a particular problem at 4096 bytes/segment (the default size, which is how I found it).

![](https://lh7-rt.googleusercontent.com/u/0/docsd/ANYlcfAPzgejyZ4bqftMHqoVCh1gQFmVREN5PuBwHSEfKNE-_l_mc_kXD46xIqJxQ2LzNGfYuqakQ1Oznt1XJhmI7scIItBL1UZisH2HNxzo74T_N9bnv4VfAPVPuw8aCSVYzRXvTS4exc0dMv7QgGFc74oEaag3iDeeAi9s6iir9aNBB4-Kt5iQ4hoL7288UzVB2UpmurBhWF5JUcLVaLE5ZCKGOnG1SsTmX-1EpYQm0_th31BufW36xbet4zZvm9ON__-W8CFI6jDsBsPy9enBMIHIF-3N7XURJRNuXZMXIG82y81KT2Xf2_9I75S3d_YnLRSCNggO1XLi5933-Lo9NOzU_Xdw4rCV0lZsg4_ZSuvDX69NLbC5uTR04ZEkhx3Inv9jxAESmGlERh35Kdu82AKfd3iKgq65nEttFWDUVI5FFT_cmWBP-dwmcVf-O622x3tLNSgO37Ab1j0uO9Sw6JzUUEfXWLLQ_xNywdW6tQ7XRY6Bc-aNtd-sp4Ddlv-DIg7xBpjsDCDHpQmhBy7jrMR40nz0tg81bwLHAQo9nlGkN6qcsGUTlrK4ff_FBdXK1jUOj01MKuxVaSd-_STcZUwuWVO9IeKHb4bmqoGsnK6amUlMbcy67uNiH7v-KdgvbHvIHEljgd90Xh8voqxRaYbvaUlJCypTQBB1VvD3oG3-56ZGVwTRDCwJF159r3nERfTEFsP8-AY6fvRrA5NPkvLXYxqyRRkvun4VFvagu5YOVe4FHlRg3TyTUiDbtM3KTAp6fPfhe7zr97pbwVgEQywi57ZjEuV3jB2qnTxoUvr1DqCcNKWDjjCTPxdxU8xOcUmICy4JeziYtUJm79Nxzcje6rbdgIcBRQV1pj_jlOQ1ZC4HoQNkT6b_ILxvVdqCpfGQoEVFywsNHx-J9HCv)

(I don’t understand why the contiguous stack impl is asymptotically better.)