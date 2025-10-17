Go Fan In and Fan Out is a concurrency paradigm where inputs from several sources get converged (multiplexed) into a single stream or inputs from one source are streamed to multiple pipelines. In simple words , this paradigm can be thought of as producer and consumer architecture , where we have multiple producers sending input to a single consumer or a single producer sending inputs to multiple consumers.

This article introduces you to Go Fan in Fan Out making use of [goroutines and channels](https://www.golinuxcloud.com/golang-concurrency/ "Golang Concurrency Explained with Best Practices"). To make good sense of this article, you should be familiar with goroutines and Go channels. To learn more about goroutines and go channels , read the  [Goroutines](https://www.golinuxcloud.com/goroutines-golang/) and [Go channels](https://www.golinuxcloud.com/go-channels/) articles respectively.

Before we get started, let us refresh our minds on what channels are.

1. Channels facilitate communications between Goroutines.
2. Sending and receiving in channels is a blocking operation.
3. Data cannot be sent to a closed channel.
4. The comma ok idiom is used to receive from a closed channel.
5. There can be more than one sender and receiver and vice versa in the same program.
6. Ranging over inputs from channels will stop only when the channel is closed.

Fan Out Fan In concurrency paradigm is made up of a pipeline which is a series of stages(Goroutines running the same function). These Goroutines are connected by Go channels. Inputs will be received via the inbound channels. These inputs will be transformed and sent via outbound channels.

### Fan Out

Fan out is used when multiple functions read from the same channel. The reading will stop only when the channel is closed. This characteristic is often used to distribute work amongst a group of workers to parallelize the CPU and I /O. Consider a function called generator that has a goroutine that iterates through  numbers 0 to 5. For each iteration a number is sent into a receiving channel and stored there temporarily. The generator function then returns a channel storing the numbers.

**Example**

go

```go
package main
 
import (
   "fmt"
   "time"
)
 
func generator(nums ...int) <-chan int {
   out := make(chan int)
   go func() {
       for _, n := range nums {
           out <- n
       }
   }()
 
   return out
}
func main() {
   fmt.Println("Start Fan Out ")
   c1 := generator(1, 2, 3)
   c2 := generator(4, 5, 6)
 
   go func() {
       for num := range c1 {
           fmt.Println(num)
       }
   }()
   go func() {
       for num := range c2 {
           fmt.Println(num)
       }
   }()
 
   time.Sleep(time.Second * 2)
}
```

**Explanation**

In the preceding example, we define a function called `generator()` that declares a Go channel called `out` of type int. After channel declaration we use a goroutine that loops through a list of integers.For each iteration, we send the current integer in the loop into the `out channel`. After the iteration is over, we return the receiving channel back to the caller. In the main function, we call the `generator()` function two times and pass it integer values. We then spin a goroutine that ranges over the values in the channels and print out the values. We then force the main goroutine to wait the goroutine for 1 second to finish executing by calling the `time.Sleep(time.Second*1)`. Without the wait feature, the main goroutines will exit the program without allowing the goroutine to finish executing.

**Output**

bash

$ go run main.go
Start Fan Out
1
2
3
4
5
6

### Data transformation

With regards to producer and consumer architecture, our architecture needs data transformation before sending the output to the consumers. We really do not need the data transformation part. We are just demonstrating common use cases when working with the Fan In Fan Out pattern. The flow of input will be `generator() → cube() → consumer()`. Please note that in our case the generator is our producer and the `main()` function is our consumer. The `cube()` function is responsible for data transformation.

**Example**

go

```go
func cube(in <-chan int) <-chan int {
   out := make(chan int)
   go func() {
       for n := range in {
           out <- n * n * n 
       }
   }()
   return out
}
```

**Explanation**

In the preceding example, we define a function called `cube()` . `The cube()` function takes a single channel as the input parameter. The `cube()` function declares an `out channel` that will pass the transformed data to the consumer. The cube() function spins a goroutine that iterates through values from the in channel that was passed as a parameter. For each iteration we calculate the cube `(n * n * n)` and pass the results to the out channel.

### Fan In

Is used when a single function reads from multiple inputs and proceeds until all are closed . This is made possible by multiplexing the input into a single channel.

In the next example, we write code that merges (multiplex) inputs from several channels into one channel.

**Example**

go

```go
func merge(in ...<-chan int) <-chan int {
   var wg sync.WaitGroup
   out := make(chan int)
 
   output := func(c <-chan int) {
       for n := range c {
           out <- n
       }
       wg.Done()
   }
   wg.Add(len(in))
   for _, c := range in {
       go output(c)
   }
   go func() {
       wg.Wait()
   }()
   return out
}
```

**Explanation**

The merge function is responsible for fanning in inputs from the `cube()` function. It takes an array of channels of type int. In the `merge()` function, we initialize a [waitgroup](https://www.golinuxcloud.com/golang-waitgroup/ "Golang WaitGroup Complete Tutorial [Beginners Guide]") using `var wg sync.Waitgroup`. The wait group is responsible for ensuring that all goroutines execute up to the end. We also initialize a channel `out` of type int that will receive all the inputs from different channels. This is where multiplexing(converging inputs)will take place. We then define an anonymous function that  iterates over inputs from a single channel and passes them into the out channel.

We then loop through  all the channels passed into the `merge()` function and then spin `output()` goroutines that take a channel from the loop. At this point, we are converging all the inputs from all the channels into the out channel. At the end we spin a separate goroutine that waits for all goroutines to return and finally return the `out` channels holding all the converged inputs.

We can now bring everything together  in the main function. The `generator()` function is the producer. It generates multiple numbers and sends them to an out channel. The `cube()` function receives inputs from the `out` channel from the `generator()` function, transforms it by getting the cube `( n* n*n)`. The cube() function sends the transformed inputs into another `out` channel. The `merge()` function merges inputs from several channels from the `cube()` function into one `out` channel.

**Example**

go

```go
package main
 
import (
   "fmt"
   "sync"
   "time"
)
 
func generator(nums ...int) <-chan int {
   out := make(chan int)
   go func() {
       for _, n := range nums {
           out <- n
       }
   }()
 
   return out
}
 
func cube(in <-chan int) <-chan int {
   out := make(chan int)
   go func() {
       for n := range in {
           out <- n * n * n
       }
   }()
   return out
}
 
func merge(in ...<-chan int) <-chan int {
   var wg sync.WaitGroup
   out := make(chan int)
 
   output := func(c <-chan int) {
       for n := range c {
           out <- n
       }
       wg.Done()
   }
 
   wg.Add(len(in))
   for _, c := range in {
       go output(c)
   }
 
   go func() {
       wg.Wait()
   }()
 
   return out
 
}
func main() {
   fmt.Println("Start Fan Out ")
   // Producer : Fan Out
   c1 := generator(1, 2, 3)
   c2 := generator(4, 5, 6)
 
   // Transformer
   cube1 := cube(c1)
   cube2 := cube(c2)
 
   // Fan In
   out := merge(cube1, cube2)
   go func() {
       for n := range out {
           fmt.Println(n)
       }
   }()
 
   time.Sleep(time.Second * 2)
}
```

**Output**

bash

$ go run main.go
Start Fan Out
1
8
64
125
27
216

## Summary

In this article we learn about the Go Fan In Fan Out concurrency pattern. This pattern consists of a pipeline which is a series of stages (goroutines) connected by channels. Each stage  can have any number of inbound and outbound channels.

## References

[https://go.dev/blog/pipelines](https://go.dev/blog/pipelines)