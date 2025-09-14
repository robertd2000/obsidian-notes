https://blog.devtrovert.com/p/select-and-for-range-channel-i-bet

#### Concurrency Series

- [Goroutines 102: A Basic Walkthrough](https://blog.devtrovert.com/p/goroutines-think-you-know-go-basics)
    
- [Go Channels Explained: More than Just a Beginner’s Guide.](https://blog.devtrovert.com/p/go-channels-explained-more-than-just)
    
- Select & For Range Channel in Go: Breaking Down
    
- [Goroutine Scheduler Revealed: Never See Goroutines the Same Way Again](https://blog.devtrovert.com/p/goroutine-scheduler-revealed-youll)
    

In our previous discussion, we covered five specific channel types in Go: [buffered, unbuffered, directional, nil and closed channels](https://blog.devtrovert.com/p/go-channels-explained-more-than-just).

If you missed that discussion, it’s important to note that we identified these types to provide a detailed understanding. Yet, at the most basic level, Go channels are primarily either buffered or unbuffered.

Now, we’re focusing on `select` and `for range` in Go.

### **1. Understanding select{}**

Basically, the `select` statement provides a mechanism for a goroutine to wait on multiple channels using the case statement and its main job is to execute the first case that is ready.

```go
select {
  case msg1 := <-ch1:
    fmt.Println("Received", msg1)
  case msg2 := <-ch2:
    fmt.Println("Received", msg2)
}
```

#### **a. How it works?**

In the example above, the `select` statement waits for either `ch1` or `ch2` to deliver data. When one of the channels is ready, the corresponding case executes.

> _**“What happens if, by some chance, multiple cases are ready simultaneously?”**_

That’s where things get interesting, select doesn’t prioritize any case over others, instead, it makes an unbiased, random choice if more than one case is ready.

But.

There’s another aspect to consider, if a `default` case is included in the select statement and none of the channels are ready, then the select does not wait, it quickly executes the default case:

```go
select {
  case msg1 := <-ch1:
    fmt.Println("Received", msg1)
  case msg2 := <-ch2:
    fmt.Println("Received", msg2)
  default:
    fmt.Println("No activity")
}
```

To summarize the behavior of the select statement, we can note the following:

- If a case in the select statement becomes ready, it gets executed.
    
- In situations where multiple cases are ready, a random one is chosen for execution.
    
- And, if no cases are ready to go, but with a `default` case in place, the `default` takes precedence. Without a `default`, the `select` statement puts the goroutine on hold.
    

Get weekly insights on Go and System Design to enhance your reading routine:

Subscribe

#### **b. Handling closed channels**

> _**“Hold on, what occurs if a channel closes while the select statement is at play?”**_

From our earlier discussion, we know that: “_sending data to a closed channel will cause a panic, while receiving data from a closed channel will return the zero value of the channel’s type if no values remain_”.

Take a look at this example:

```go
ch1 := make(chan int)
close(ch1)

select {
  case msg1 := <-ch1:
    fmt.Println("Received", msg1)  // This line will execute.
  case msg2 := <-ch2:
    fmt.Println("Received", msg2)
}
```

After you close `ch1`, the `case msg1 := <-ch1` line will execute right away, returning the zero value of the channel’s type, consistent with what we know about channels right?

But there’s a question: how can we tell if the returned `msg1` is a zero value due to a closed channel or an intended value? We can refine our approach for better clarity:

```go
select {
  case msg1, ok := <-ch1:
    if ok {
      fmt.Println("Received", msg1)
    } else {
      fmt.Println("Channel closed!")
    }
  case msg2 := <-ch2:
    fmt.Println("Received", msg2)
}
```

With this approach, we can not only receive data but also determine the state of our channels.

#### **c. Timeout Trap**

When you need to handle timeouts, the `select` statement in Go makes it easy.

You can make sure the operation doesn’t get stuck by using `time.After`. This function lets you move on after waiting for a certain amount of time, instead of waiting forever for something to happen:

```go
select {
  case msg1 := <-ch1:
    fmt.Println("Received", msg1)
  case <-time.After(2 * time.Second):
    fmt.Println("Timeout")
}
```

Here, if `ch1` doesn’t receive any message within 2 seconds, the case associated with `time.After` activates, acting as our timeout mechanism.

However, there’s a nuance to be mindful of with `time.After`.

If `ch1` becomes ready before the timeout, the `select` exits, but the timer keeps running in the background. When it does expire, the timer sends the timeout value to its channel, even though we’re not waiting on it.

This might not seem problematic at first, but it can affect memory usage since this value stays in memory until its timer runs out.

To handle this more efficiently, it’s recommended to use `time.NewTimer`:

```go
timer := time.NewTimer(2 * time.Second)

select {
  case msg1 := <-ch1:
    fmt.Println("Received", msg1)
    timer.Stop()
  case <-timer.C:
    fmt.Println("Timeout")
}

fmt.Println("After select statement")
```

By using this method, if `ch1` sends a message before the timer completes, we can immediately stop the timer with `timer.Stop()`, freeing up its associated resources.

> _**“It’s only a short period, is it that significant?”**_

On its own, a few seconds might not seem like much, but in real-world situations, where the waiting time could be much longer and combined with loops (worst case), these timers can add up, using more memory than necessary.

While these timers eventually free up their resources, it’s always good practice to be efficient and mindful of potential issues.

#### **d. Combining for and select{}.**

When we use a for loop together with a `select` statement, we create a method for continually checking multiple channels:

```go
for {
  select {
  case msg1 := <-ch1:
    fmt.Println("Received from ch1:", msg1)
  case msg2 := <-ch2:
    fmt.Println("Received from ch2:", msg2)
  case <-time.After(1 * time.Second):
    fmt.Println("Waiting...")
  }
}
```

Here, our application enters a never-ending for loop, and within this loop, the `select` statement is waiting on several things at once: the channels `ch1` and `ch2`, and also a timer.

However, there’s an issue if one of the channels, like `ch1`, is closed, this closed channel stops being a blockage and starts sending endless default values, cluttering the output with repetitive messages:

```
Received from ch1: 0
Received from ch1: 0
Received from ch1: 0
Received from ch1: 0
…
```

To manage this, we use a technique where we check the status of the received message, and if `ok` is false, it means the channel has closed.

```go
for {
  select {
    case msg1, ok := <-ch1:
      if !ok {
        fmt.Println("ch1 is closed. Exiting loop.")
        break
      }
      fmt.Println("Received from ch1:", msg1)
    case msg2, ok := <-ch2:
      if !ok {
        fmt.Println("ch2 is closed. Exiting loop.")
        break
      }
      fmt.Println("Received from ch2:", msg2)
    case <-time.After(1 * time.Second):
      fmt.Println("Waiting...")
    }
}
```

But, there’s a problem here, will it stop as we expect?

Unfortunately, it doesn’t. The `break` here doesn’t quit the for loop, it only exits the `select`, and the loop goes on without end.

The solution involves using something called “labels” which we will discuss next.

#### **e. Label**

Go allows loops to have labels and with these, you can specify exactly which loop you intend to ‘break’ out of, particularly useful when dealing with nested structures.

```go
OuterLoop:
  for {
    select {
      case msg1, ok := <-ch1:
        if !ok {
          fmt.Println("ch1 is closed. Exiting loop.")
          break OuterLoop // This exits the loop named OuterLoop
        }
        fmt.Println("Received from ch1:", msg1)
      case msg2, ok := <-ch2:
        if !ok {
          fmt.Println("ch2 is closed. Exiting loop.")
          break OuterLoop // Same here
        }
        fmt.Println("Received from ch2:", msg2)
      case <-time.After(1 * time.Second):
        fmt.Println("Waiting...")
      }
  }
```

Here, `OuterLoop` is the label.

The `break OuterLoop` statement tells the application to exit the `OuterLoop` when it runs and this label helps in directing the flow precisely, avoiding any unintended continuations.

#### f. Non-blocking Send Trick

So far, we’ve been primarily focused on receiving from channels, like in the examples

```go
case msg1, ok := <-ch1:
// ...
case msg2, ok := <-ch2:
// ...
```

These are pretty straightforward, right?

But, what happens if we flip the script and instead of receiving, we start sending values into the channels within a select? Consider this different approach:

```go
select {
  case ch1 <- 1:
    fmt.Println("Sent 1 to ch1")
  case ch2 <- 2:
    fmt.Println("Sent 2 to ch2")
  default: // <---- 
    fmt.Println("Neither channel was ready for sending")
}
```

Here’s where things get a bit more nuanced.

When sending to a channel in a select statement, if the channel is not ready to receive the value (for example, if there’s no corresponding goroutine ready to receive on the other end), the select will not block waiting for it to become ready.

If no channels are ready and there’s a **default** **case**, it will execute that.

This behavior is particularly useful when you want to avoid blocking in situations where you’re not sure if the receiver is ready.

> _**“Is there any real-world application?”**_

Yes.

There is a package called `errgroup` (from the standard library) that uses this trick to implement its waiting strategy.

### **2. Select Control with Nil Channel**

Now, suppose you’re dealing with two channels, ch1 and ch2. If ch1 closes, you might want to keep the loop alive for ch2 without getting repeated messages.

Here’s how you can do it by setting a channel to `nil`:

```go
OuterLoop:
  for {
    select {
      case msg1, ok := <-ch1:
        if !ok {
          fmt.Println("ch1 is closed. Switching off.")
          ch1 = nil
        }
        fmt.Println("Received from ch1:", msg1)
      case msg2, ok := <-ch2:
        if !ok {
          fmt.Println("ch2 is closed. Exiting loop.")
          ch2 = nil
        }
        fmt.Println("Received from ch2:", msg2)
      case <-time.After(1 * time.Second):
        fmt.Println("Waiting...")
        
        if (ch1 == nil) && (ch2 == nil) {
          fmt.Println("Both channels are closed. Exiting loop.")
          break OuterLoop
        }
    }
  }
```

So, what’s happening here?

Initially, both `ch1` and `ch2` are active, if one closes, we make that channel `nil`, meaning it won’t receive any more data (block forever). We use `time.After` to check regularly, and if there’s no active channel left (both are `nil`), we exit the loop.

By doing this, you avoid keeping the loop running for no reason, and you manage channel activity in a clean, efficient way.

> _**“What if I don’t set ch1 to nil?”**_

Without setting ch1 to nil, the output continuously displays `Received from ch1`, and it keeps repeating because it’s no longer receiving meaningful data, just the default over and over.

This continues until ch2 closes and timeout happens.

### **3. For Range**

Using `for range` with channels in Go is quite handy, instead of repeatedly checking if there’s a new value in the channel, now you can just sit back and let the loop handle it.

Uniquely, the `for range` used with a channel yields only one value, different from other iterations (like `for k, v := range m`):

```go
ch := make(chan int)

go func() {
  for i := 0; i < 10; i++ {
    ch <- i
  }

  close(ch) // Don't forget this step!
}()

for value := range ch {
  fmt.Println("Received:", value)
}
```

Here, the loop `for value := range ch` will keep pulling data from the `ch` channel until you close it.

There is a common mistake, if you forget to close the channel, the loop will just keep waiting and it’ll be stuck, waiting for new values forever (deadlock). So, if you’re running a bunch of goroutines, this mistake can eat up memory pretty fast.

What occurs if you close a buffered channel that still contains values, does the loop `for value := range ch` continue to retrieve data from the channel?

```go
ch := make(chan int, 3)
ch <- 1
ch <- 2
ch <- 3

close(ch)

for value := range ch {
  fmt.Println("Received:", value)
}
```

Here’s what happens:

```
Received: 1
Received: 2
Received: 3
```

Indeed, the loop continues to function and pulling out every value in the buffer, this characteristic ensures that no values are stranded within a closed channel.

---

It was late and without coffee, I almost started dreaming while writing this post. If you spot something off, it’s probably because my brain was craving sleep or caffeine, feel free to point it out, it helps me stay on track.