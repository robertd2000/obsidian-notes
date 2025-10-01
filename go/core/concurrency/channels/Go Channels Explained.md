https://blog.devtrovert.com/p/go-channels-explained-more-than-just

#### Concurrency Series

- [Goroutines 102: A Basic Walkthrough](https://blog.devtrovert.com/p/goroutines-think-you-know-go-basics)
    
- Go Channels Explained: More than Just a Beginner’s Guide.
    
- [Select & For Range Channel in Go: Breaking Down](https://blog.devtrovert.com/p/select-and-for-range-channel-i-bet)
    
- [Goroutine Scheduler Revealed: Never See Goroutines the Same Way Again](https://blog.devtrovert.com/p/goroutine-scheduler-revealed-youll)
    

After discussing Goroutines, diving into Goroutines vs OS Threads, examining `MAXPROCS` in the previous post, [Goroutines: Think You Know Go Basics? Think Again](https://blog.devtrovert.com/p/goroutines-think-you-know-go-basics).

We are now ready to talk about Channels, which act as synchronization points.

### **1. Channel**

Imagine a channel as a simple pipe, this pipe connects different goroutines, allowing them to talk to each other by sending data into one end of the pipe and receiving it from the other.

Why exactly do we use channels?

Channels give us a safe way for goroutines to communicate and also to keep their actions in sync.

While the usual method of using locks to control access to data can be tricky and might cause deadlocks, channels make managing concurrency straightforward. They follow a simple idea: _‘Don’t communicate by sharing memory; share memory by communicating.’_

But before we delve deeper, let’s first discuss the basic syntax of a channel.

#### **Creating Channels**

```go
var c1 chan int
ch := make(chan int)
```

Here, c1 is declared as a channel that deals with integers, yet it’s uninitialized, it holds a `nil` value.

In contrast, `ch` is an unbuffered channel for integers.

#### **Channel Basic Operations**

```go
 ch <- 42      // sends 42 to channel ch
 value := <-ch // receive from channel ch
 close(ch)
```

We’ll dig deeper into buffered channels later, but for now, let’s stick with our unbuffered channel `ch`.

When you write `ch <- 42`, the goroutine takes a pause, waiting until a receiver is ready. And it works the same when you’re on the receiving end of a channel.

Here’s a simple example to show what’s happening:

```go
func main() {
  ch := make(chan int) // Creating an unbuffered channel.

  // A goroutine to send a value.
  go func() {
    fmt.Println("Ready to send 42...")
    ch <- 42
    fmt.Println("42 is sent.")
  }()
  
  // Waiting to get the value from the channel.
  fmt.Println("Waiting for value from channel...")
  val := <-ch
  fmt.Println("Value received:", val)
}
```

Running the above example will give us the output like so:

```
Waiting for value from channel...
Ready to send 42...
42 is sent.
Value received: 42
```

[

![](https://substackcdn.com/image/fetch/$s_!1rZ7!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0e106cb4-3110-4b3c-9718-0caa226a2ca8_750x850.png)



](https://substackcdn.com/image/fetch/$s_!1rZ7!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0e106cb4-3110-4b3c-9718-0caa226a2ca8_750x850.png)

Channel example 1 (source. blog.devtrovert.com)

Here’s a little breakdown:

- The main goroutine sets up an unbuffered channel and schedules a goroutine to send a value to this channel.
    
- When the main goroutine hits the line `val := <-ch`, it stops and waits until a value is sent to `ch`.
    
- The scheduled goroutine sends 42 to the channel.
    
- The main goroutine gets the value from the channel and displays it.
    

In case you’re wondering why “_Waiting for value from channel…_” shows up before “_Ready to send 42…_”, feel free to revisit our [previous post about Goroutines](https://blog.devtrovert.com/p/goroutines-think-you-know-go-basics).

> _**“But what happens when we close a channel?”**_

Invoking `close(ch)` gives a clear signal: this channel is no longer in use, if you try to send a value through it, the application will panic.

But interestingly, trying to receive from a closed channel won’t trigger a panic. Instead, it provides the zero value of the channel’s type (assuming it’s unbuffered).

A few things to note:

- Closing a channel is a “definitive” act, there’s no turning back or reopening it.
    
- Sadly, Go doesn’t offer a built-in method to check if a channel is closed. You’ll only discover it’s closed when you try to read from it and you get a zero value when it’s void of data.
    

```go
func main() {
  ch := make(chan int)
  close(ch)

  v, ok := <-ch

  fmt.Println(v, ok) // 0 false
}

// If the second value (a boolean) is false, 
// the channel is [closed] and [has no values left to receive]
```

That wraps up the basics, hope you’re ready for a deeper dive into some more engaging content.

Get weekly insights on Go and System Design to enhance your reading routine:

Subscribe

### **2. Channel Types**

We’re diving into two main channel types in Go: **unbuffered** and **buffered** channels.

While some may argue there are more types, particularly when looking at **directional** channels, I view those as merely syntactical variations, limited by the compiler

Rather than classifying channel types by certain properties, we’ll talk about them through their behavior, including unbuffered, buffered, directional, nil channels, and closed channels.

#### **a. Unbuffered Channel**

You might recognize this type from earlier examples.

An unbuffered channel is what you get when you initialize a channel without specifying a size, or with a size of 0.

```go
uch1 := make(chan int)
uch2 := make(chan int, 0)
```

Now, let’s discuss their characteristics:

**Immediate Blocking:  
**When a value is sent on an unbuffered channel (e.g., `uch1 <- 42`), the send operation is blocked until another goroutine executes a corresponding receive (e.g., `value := <-uch1`).

**Synchronization:  
**This blocking behavior makes unbuffered channels naturally synchronize.

So, if one goroutine sends data and another receives it, the operations are synchronized at the point of data exchange, known as _“synchronous communication”_.

**Deadlock:  
**If a goroutine sends data to an unbuffered channel, there must be another goroutine to receive from the channel, and vice versa, to avoid deadlock.

If not, the program will fatal (not panic).

```go
ch := make(chan int)

fmt.Println("Receiving from channel…")
val := <-ch // fatal error: all goroutines are asleep - deadlock!

fmt.Println("Received:", val)
```

#### **b. Buffered Channel**

Buffered channels introduce an interesting twist… they don’t block immediately upon sending a value. Instead, they can accommodate a specific number of values before they get blocked.

Think of it like a mini storage space within the channel.

```go
func main() {
    ch := make(chan int, 3)
    go func() {
        fmt.Println("Goroutine: Waiting for a value from the channel...")
        fmt.Printf("Goroutine: Got the value %d from the channel.\n", <-ch)
    }()

    // Populating the buffered channel to its limit.
    ch <- 1
    ch <- 2
    ch <- 3
    fmt.Println("Buffer is now full.")
    ch <- 4
    fmt.Println("This line prints after the 4th value gets through.")
}
```

```
Buffer is now full.
Goroutine: Waiting for a value from the channel...
Goroutine: Got the value 1 from the channel.
This line prints after the 4th value gets through.
```

After pushing 3 values to the channel, we’re able to proceed to the message _“Buffer is now full”, b_ut the 4th value makes us pause until the goroutine fetches a value.

> _**“What about when we fetch data? Will it block too?”**_

Absolutely, the receiving side does block if there’s nothing to fetch.

**Capacity & Length:  
**Buffered channels introduce us to two new concepts: capacity and length.

- Capacity: The maximum number of values a channel can store.
    
- Length: The current count of values in the channel
    

Taking the earlier example, when sending the 4th value with `ch <- 4`, can you guess `len(ch)`? Try it yourself before reading on.

If you thought 3, you’re right on the mark, even when full, the channel length stays at 3, pausing the send operation until there’s space.

[

![](https://substackcdn.com/image/fetch/$s_!JFmI!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F04a41a12-9077-41a2-9409-19eabfb8b052_938x1063.png)



](https://substackcdn.com/image/fetch/$s_!JFmI!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F04a41a12-9077-41a2-9409-19eabfb8b052_938x1063.png)

Buffered channel example (source: blog.devtrovert.com)

Hence, the buffered channel’s flow in the earlier instance can be perceived as: send (1, 2, 3) -> successful send (1, 2, 3) -> full -> send (4) -> block -> receive (1) -> successful send (4).

#### **c. Directional Channel**

By default, channels are bidirectional, permitting data to flow both ways. Go provides an option to restrict them, specifying if a channel should only send or receive data.

But, in technical terms, these aren’t different ‘types’ of channels.

They simply apply a **compile-time constraint** to ensure that channels operate correctly and are used in the expected way in specific contexts.

**Send-only Channel  
**This channel can only send values, creating a clear communication that certain channels are strictly for sending data:

```go
var sendCh chan<- int = make(chan int)
v := <- sendCh // invalid operation: cannot receive from send-only channel sendCh
```

**Receive-only Channel  
**In contrast, this channel can only receive data, blocking any attempts to send data:

```go
var recvCh <-chan int = make(chan int)
recvCh <- 42 // invalid operation: cannot send to receive-only channel recvCh
```

> _**“If a channel can only send or only receive, how is it useful? Don’t we need both functionalities for it to work?”**_

A valid point.

The key reason for having directional channels is to enforce particular behaviors in different parts of the code to avoid errors and clarify its purpose, not during channel creation, but during its usage.

When you pass a send-only channel to a function, you’re clearly stating: _“This function is designed to produce data and send it, it should not be responsible for receiving data”:_

```go
// The function can only send to the channel.
func producer(ch chan<- int) {
    ch <- 42
}
// The function can only receive from the channel.
func consumer(ch <-chan int) int {
    return <-ch
}
```

At a deeper level, regardless of being send-only or receive-only, a channel is just a channel and the directionality acts more as a compile-time constraint than a runtime one.

This means the Go compiler makes sure the channel is used correctly based on the type, even though, during runtime, a send-only and a receive-only channel don’t have differences in their underlying structure from a regular bidirectional channel.

> _**“If the basic structure is the same, why bother specifying the direction?”**_

By using directional channels in specific situations, you establish safer and clearer code.

It’s a method to communicate the purpose and usage of the code to others, enhancing both individual and team development efforts by adding a layer of transparency and safety.

> _**“Can I convert back and forth?”**_

Indeed, Go allows us to convert channels, it’s simple to convert from bidirectional to unidirectional, but doing the reverse is not as straightforward because the compiler imposes certain restrictions.

Here’s a quick look at how to convert to unidirectional channels:

```go
ch := make(chan int) // Bidirectional

var chSend chan<- int = ch // Convert to send-only
var chReceive <-chan int = ch // Convert to receive-only
```

Yet, if you’re wondering about converting back, here’s a technique that employs the `unsafe` package:

```go
func main() {
  ch := make(chan int) // Bidirectional
  
  var sendCh chan<- int = ch
  
  // Convert sendCh back to bidirectional using unsafe
  bidiCh := *(*chan int)(unsafe.Pointer(&sendCh))
  
  go func() {
  bidiCh <- 42
  }()
  
  fmt.Println(<-ch) // prints 42
}
```

While this method might be useful in some situations, it’s generally recommended to be cautious when using `unsafe`, especially in critical systems where stability is key.

#### **d. Nil Channel**

Channels in Go can hold a `nil` value, and dealing with them comes with its own set of rules that might feel a bit unusual if you're navigating through Go for the first time.

**Send and Receive Actions:  
**When you try to send data through a nil channel, you might expect to face a panic.

But, instead, what happens is perpetual blocking:

```go
var ch chan int  // ch is nil initially

go func() {
    ch <- 42  // This will block indefinitely
}()

value := <-ch  // This too, will block indefinitely
```

There are no panics, no errors. So, you might wonder, is there a useful side to this?

Indeed, there are scenarios where this characteristic might be exploited intentionally to deactivate a channel selection, a tactic we’ll uncover more comprehensively in an upcoming discussion.

Although sending or receiving through a nil channel won’t raise a panic, attempting to close it will:

```go
var ch chan int  // ch is nil by default
close(ch)  // panic: close of nil channel
```

This can be a bit perplexing, especially when we discuss other types of channels in the upcoming sections.

#### **e. Closed Channel**

Closed channels in Go can be tricky, yet understanding their behaviors is quite crucial to effectively manage potential issues.

**Send Behavior  
**First, let’s talk about sending to a closed channel. Unlike a nil channel, it doesn’t get blocked forever.

Instead, it results in an **immediate panic** to prevent unintended sends after the channel has signaled no more data will be sent:

```go
ch := make(chan int)
close(ch)
ch <- 42  // "send on closed channel"
```

**Received Behavior  
**On the other hand, receiving from a closed channel is slightly more nuanced:

- If the channel buffer still holds values, they can be received as if the channel is still open.
    
- Once the buffer is empty, further receives will return the zero value for the channel’s type without blocking.
    

```go
ch := make(chan int, 3)
ch <- 1
ch <- 2
ch <- 3
close(ch)
fmt.Println(<-ch) // 1
fmt.Println(<-ch) // 2
fmt.Println(<-ch) // 3
```

If you’d like to know if a channel has been closed, you can utilize a two-value receive. If the second value (a boolean) is false, the channel is [closed] and [has no values left to receive]:

```go
ch := make(chan int, 3)
ch <- 1
close(ch)

value, ok := <-ch
fmt.Println(value, ok) // 1 true

value, ok = <-ch
fmt.Println(value, ok) // 0 false
```

Isn’t that unexpected? Sending causes a panic, while receiving doesn’t and can even retrieve values.

This design is intentional, and I’ll explain why shortly.

**Close Behavior  
**Similar to sending to a closed channel, trying to close an already closed channel also results in a panic.

```go
ch := make(chan int, 3)
close(ch)

close(ch) // panic: close of closed channel
```

> _**“How do we do to avoid these annoying issues?”**_

In various common situations, particularly in producer-consumer patterns, it is often more logical for the sending (producer) goroutine to close the channel, not the receiving (consumer) goroutine.

Why?

The producer is typically aware when all data has been sent, making it an ideal candidate to close the channel and signal to the consumers that no more data will be coming.

That’s why receiving data from a closed channel won’t trigger a panic, while trying to send data to a closed channel will.

To make things clear, let’s summarize their behavior:

[

![](https://substackcdn.com/image/fetch/$s_!npuJ!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F091ce583-f070-46bd-9bfc-1545df5b6c32_1200x675.png)

](https://substackcdn.com/image/fetch/$s_!npuJ!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F091ce583-f070-46bd-9bfc-1545df5b6c32_1200x675.png)