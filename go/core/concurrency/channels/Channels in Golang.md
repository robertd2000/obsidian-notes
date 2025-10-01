https://golangdocs.com/channels-in-golang

Channels are a medium that the [goroutines](https://golangdocs.com/goroutines-in-golang) use in order to communicate effectively. It is the most important concept to grasp after understanding how goroutines work. This post aims to provide a detailed explanation of the working of the channels and their use cases in Go.

## Golang Channels syntax

In order to use channels, we must first create it. We have a very handy function called make which can be used to create channels. A channel is dependent on the data type it carries. That means we cannot send strings via int channels. So, we need to create a channel-specific to its purpose.

Here’s how we create channels. The **chan** is a keyword which is used to declare the channel using the make function.

|   |   |
|---|---|
|1<br><br>2|`// a channel that only carries int`<br><br>`ic := make(chan` `int``)`|

To send and receive data using the channel we will use the channel operator which is `**<-**` .

|   |   |
|---|---|
|1<br><br>2|`ic <-` `42`         `// send 42 to the channel`<br><br>`v := <-ic`        `// get data from the channel`|

## Zero-value of a channel

A channel that is not initialized or zero-value is nil.

|   |   |
|---|---|
|1<br><br>2|`var ch chan` `int`<br><br>`fmt.Println(ch)`    `// <nil>`|

## Working with channels

Now, we will try sending and receiving data using channels. Let’s start by creating a basic goroutine that will send data via a channel which we will be receiving from the main goroutine.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>`)`<br><br>`func SendDataToChannel(ch chan` `int``, value` `int``) {`<br><br>    `ch <- value`<br><br>`}`<br><br>`func main() {`<br><br>    `var v` `int`<br><br>    `ch := make(chan` `int``)`     `// create a channel`<br><br>    `go SendDataToChannel(ch,` `101``)`   `// send data via a goroutine`<br><br>    `v = <-ch`                        `// receive data from the channel`<br><br>    `fmt.Println(v)`       `// 101`<br><br>`}`|

Here, in this example, we send data via a goroutine and receive data accordingly. Now, we will try sending custom data such as a struct.

## Sending custom data via channels

Custom data can be sent just like any other data type. When creating and using the channels we need to be aware of using the correct data type when creating the channel. Here is an example that sends a Person struct via a channel.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27|`package` `main`<br><br>`import` `(`<br><br>    `"fmt"`<br><br>    `// "time"`<br><br>`)`<br><br>`type Person struct {`<br><br>    `Name string`<br><br>    `Age`  `int`<br><br>`}`<br><br>`func SendPerson(ch chan Person, p Person) {`<br><br>    `ch <- p`<br><br>`}`<br><br>`func main() {`<br><br>    `p := Person{``"John"``,` `23``}`<br><br>    `ch := make(chan Person)`<br><br>    `go SendPerson(ch, p)`<br><br>    `name := (<-ch).Name`<br><br>    `fmt.Println(name)`<br><br>`}`|

## The send and receive operation

Tha channels operations are by default blocking. That means when we use any of the send or receive operation the channels blocks unless the work is done. Thus allowing them to be synchronized.

## Using directional channels

Channels can be unidirectional. That means channels can be declared such that the channel can only send or receive data. This is an important property of channels.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15|`package` `main`<br><br>`func f(ch chan<-` `int``, v` `int``) {`<br><br>    `ch <- v`<br><br>`}`<br><br>`func main() {`<br><br>        `// send-only channel`<br><br>    `ch := make(chan<-` `int``)`<br><br>    `go f(ch,` `42``)`<br><br>    `go f(ch,` `41``)`<br><br>    `go f(ch,` `40``)`<br><br>`}`|

In the code above, we use a channel which is a send-only channel. That means data can only be sent into it but when we try receiving any data from the channel it produces errors.

The syntax is as follows:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5|`ch := make(chan<- data_type)`        `// The channel operator is after the chan keyword`<br><br>                                    `// The channel is send-only`<br><br>`ch := make(<-chan data_type)`        `// The channel operator is before the chan keyword`<br><br>                                    `// The channel is receive-only`|

## Closing a channel

A channel can be closed after the values are sent through it. The close function does that and produces a boolean output which can then be used to check whether it is closed or not.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26|`package` `main`<br><br>`import` `"fmt"`<br><br>`func SendDataToChannel(ch chan string, s string) {`<br><br>    `ch <- s`<br><br>    `close(ch)`<br><br>`}`<br><br>`func main() {`<br><br>    `ch := make(chan string)`<br><br>    `go SendDataToChannel(ch,` `"Hello World!"``)`<br><br>    `// receive the second value as ok`<br><br>    `// that determines if the channel is closed or not`<br><br>    `v, ok := <-ch`<br><br>    `// check if closed`<br><br>    `if` `!ok {`<br><br>        `fmt.Println(``"Channel closed"``)`<br><br>    `}`<br><br>    `fmt.Println(v)` `// Hello World!`<br><br>`}`|

## Using a loop with a channel

A range loop can be used to iterate over all the values sent through the channel. Here is an example showing just that.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22|`package` `main`<br><br>`import` `"fmt"`<br><br>`func f(ch chan` `int``, v` `int``) {`<br><br>    `ch <- v`<br><br>    `ch <- v *` `2`<br><br>    `ch <- v *` `3`<br><br>    `ch <- v *` `7`<br><br>    `close(ch)`<br><br>`}`<br><br>`func main() {`<br><br>    `ch := make(chan` `int``)`<br><br>    `go f(ch,` `2``)`<br><br>    `for` `v := range ch {`<br><br>        `fmt.Println(v)`<br><br>    `}`<br><br>`}`|

This program produces an output shown below:

![Go Channel Range Loop](https://golangdocs.com/wp-content/uploads/2020/03/go-channel-range-loop.png)

Go Channel Range Loop

As we can see, that the loop is done over all the values the channel sends. The program outputs as expected. The channel also should be closed after sending the values.