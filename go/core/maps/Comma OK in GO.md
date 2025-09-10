https://dev.to/saurabh975/comma-ok-in-go-l4f

Have you ever come across a statement like this?  

```go
func main(){
  i:= "hello world"
  n, ok := i.(int)
   if ok {
    fmt.Println("n :", n)
   } else {
    fmt.Println("not an int")
   }
}
```

Well, there’s no Try Catch block in Go. Coming from Java/Scala background, it felt weird at first but then GO grows on you, and then voila…you’re handling errors like a pro.

Comma ok is one such way of error handling. It’s used in 4 different scenarios:

1. Test for error on a function return
2. Testing if a key-value item exists in a Go map
3. Testing if an interface variable is of a certain type
4. Testing if a channel is closed

Let’s have a look at each one with an example

## [](https://dev.to/saurabh975/comma-ok-in-go-l4f#1-test-for-error-on-a-function-return)1. Test for error on a function return

```go
func main() {
 if ok, err := isEven(3); ok {
  fmt.Println("it's an even number")
 }else{
  fmt.Println(err)
 }
}

func isEven(n int)(bool, error){
 if n & 1 == 1{
  return false, fmt.Errorf("it's an odd number")
 }
 return true, nil
}
```

The function isEven() takes an integer as input and checks if it’s even or not. The return type of isEven() is boolean and error. We store the boolean value in ok and use it in conditional statements.

## [](https://dev.to/saurabh975/comma-ok-in-go-l4f#2-testing-if-a-keyvalue-item-exists-in-a-go-map)2. Testing if a key-value item exists in a Go map

```go
func main(){
   var stocks map[string]float64
   sym := "TTWO"
   price := stocks[sym]
   fmt.Printf("1. %s -> $%.2f\n", sym, price)

   if price, ok := stocks[sym]; ok {    // using comma ok to check if key exists
    fmt.Printf("2. %s -> $%.2f\n", sym, price)
   } else {
    fmt.Printf("2. %s not found\n", sym)
   }

   stocks = make(map[string]float64)   // It's importatnt to initialize the map before adding values. Else it's a panic situation
   stocks[sym] = 123.4
   stocks["APPL"] = 345.6

   if price, ok := stocks[sym]; ok {
    fmt.Printf("3. %s -> $%.2f\n", sym, price)
   } else {
    fmt.Printf("3. %s not found\n", sym)
   }
}
```

```
OUTPUT:
1. TTWO -> $0.00
2. TTWO not found
3. TTWO -> $123.40
```

Let’s understand what’s happening in the code.

At first, we declare a map stocks with (key, value) type as (string, float). We try to get the value for key=TTWO, as there’s no key that matches TTWO, the default value of float, i.e. 0.00 is returned.

> Note that there’s no error that was returned. We actually don’t know if 0.00 is the assigned value or the default value.

This is where comma ok comes to the rescue. GO compiler is smart enough to understand what to return when the comma ok is added so in the line price, ok := stocks[sym], ok is assigned false, helping us with the dilemma.

## [](https://dev.to/saurabh975/comma-ok-in-go-l4f#3-testing-if-an-interface-variable-is-of-a-certain-type)3. Testing if an interface variable is of a certain type

```go
func main(){
  i := "hello world"
  if n, ok := i.(int); ok{
    fmt.Println("It's an int")
  }else{
    fmt.Println("Not an int")
  }
}
```

This is just a slightly modified version of the code you saw above, to match the pattern we are following. That’s all 😉.  
Notice how intelligent the GO-compiler is? It knows in what context the comma ok is being used.

## [](https://dev.to/saurabh975/comma-ok-in-go-l4f#4-testing-if-a-channel-is-closed)4. Testing if a channel is closed

> GO channel is a medium through which a goroutine communicates with another goroutine and this communication is lock-free  

```go
func main() {
 ch := make(chan string)    //initializing a channel
 go func() {
  for i := 1; i <= 3; i++ {
   msg := fmt.Sprintf("Current num is #%v", i)
   ch <- msg
  }
  close(ch) //closing the channel lets range know exactly ho much data is there
 }()

 for msg := range ch {
  fmt.Println("received message : ", msg)
 }

 msg := <-ch
 fmt.Printf("Message from a closed channel : %#v\n", msg)

 msg, ok := <-ch  // recieving value through a channel with comma ok
 fmt.Printf("Message : %#v, (was it closed? -> %#v)\n", msg, !ok)
}
```

```
OUTPUT : 
received message :  Current num is #1
received message :  Current num is #2
received message :  Current num is #3
Message from a closed channel : ""
Message : "", (was it closed? -> true)
```

We create a go routine, which pushes 3 integers into the channel ch and closes it before exiting the routine. The for loop consumes the data from channel ch. Then we try to assign the next message in the channel to msg. Since it’s empty we get the output as an empty string.

> Channels don’t run into the panic situation if there's nothing to consume. They simply return a default value. But you cannot add any value into a closed channel. It gives rise to a panic situation.

We don’t really know if the value being consumed is the actual value or the default value. This is where comma ok comes to the rescue.

