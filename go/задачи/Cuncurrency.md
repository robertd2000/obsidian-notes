
https://www.youtube.com/watch?v=3oGmaIGPv54&t=1943s

## 1 - отрефакторить

условие
```go

package main

import (
    "sync"
)

type Stack struct {
    mutex sync.Mutex
    data  []string
}

func NewStack() Stack {
    return Stack{}
}

func (b *Stack) Push(value string) {
    b.mutex.Lock()
    defer b.mutex.Unlock()
    b.data = append(b.data, value)
}

func (b *Stack) Pop() {
    if len(b.data) < 0 {
        panic("pop: stack is empty")
    }
    b.mutex.Lock()
    defer b.mutex.Unlock()
    b.data = b.data[:len(b.data)-1]
}

func (b *Stack) Top() string {
    if len(b.data) < 0 {
        panic("top: stack is empty")
    }
    b.mutex.Lock()
    defer b.mutex.Unlock()
     return b.data[len(b.data) - 1]
}

var stack Stack

func producer() {
    for i := 0; i < 1000; i++ {
        stack.Push("message")
    }
}

func consumer() {
    for i := 0; i < 10; i++ {
        _ = stack.Top()
        stack.Pop()
    }
}

func main() {
    producer()

    wg := sync.WaitGroup{}
    wg.Add(100)

    for i := 0; i < 100; i++ {
        go func() {
            defer wg.Done()
            consumer()
        }()
    }

    wg.Wait()
}
    
```
решение
```go

package main

import (
    "sync"
)

type Stack struct {
    mutex sync.Mutex
    data  []string
}

func NewStack() *Stack {
	return &Stack{	}
}

func (b *Stack) Push(value string) {
    b.mutex.Lock()
    defer b.mutex.Unlock()
    b.data = append(b.data, value)
}

func (b *Stack) Pop() {
    b.mutex.Lock()
    defer b.mutex.Unlock()

    if len(b.data) < 0 {
        panic("pop: stack is empty")
    }
    
    val := b.data[len(b.data) - 1]
    b.data = b.data[:len(b.data)-1]
    
    return val
}

var stack Stack

func producer() {
    for i := 0; i < 1000; i++ {
        stack.Push("message")
    }
}

func consumer() {
    for i := 0; i < 10; i++ {
        stack.Pop()
    }
}

func main() {
    producer()

    wg := sync.WaitGroup{}
    wg.Add(100)

    for i := 0; i < 100; i++ {
        go func() {
            defer wg.Done()
            consumer()
        }()
    }

    wg.Wait()
}

```


## 2.

```go

package main

import "sync"

type Buffer struct {
    mtx  sync.Mutex
    data []int
}

func NewBuffer() *Buffer {
    return &Buffer{}
}

func (b *Buffer) Add(value int) {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    b.data = append(b.data, value)
}

func (b *Buffer) Data() []int {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    return b.data 
}

```

решение

```go

package main

import "sync"

type Buffer struct {
    mtx  sync.Mutex
    data []int
}

func NewBuffer() *Buffer {
    return &Buffer{}
}

func (b *Buffer) Add(value int) {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    b.data = append(b.data, value)
}

func (b *Buffer) Data() []int {
    b.mtx.Lock()
    defer b.mtx.Unlock()
    return b.data // срез отдается пользователю
    // можно вернуть копию
    // или можно сделать цикл и так данные отдавать
}

```

## 3

```go

package main

import (
    "sync"
)

type Cache struct {
    mutex sync.Mutex
    data  map[string]string
}

func NewCache() *Cache {
    return &Cache{
        data: make(map[string]string),
    }
}

func (c *Cache) Set(key, value string) {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    c.data[key] = value
}

func (c *Cache) Get(key string) string {
    c.mutex.Lock()
    defer c.mutex.Unlock() 
    if c.Size() > 0 { // при вызове c.Size() опять вызовется мьютекс и может повиснуть все из-за рекурсивной блокировки
    // нужно использовать sizeLocked - вот так
    // if c.sizeLocked() > 0
        return c.data[key]
    }
    return ""
}

func (c *Cache) Size() int {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    
    return c.sizeLocked()
}

// можно так

func (c *Cache) sizeLocked() int {
    return len(c.data)
}

```


## 4

```go

package main

import "sync"

type Data struct {
	sync.Mutex // из-за встраивания структуры мьютекса пользвоатель получает доступ к его методам
	// m sync.Mutex
	values []int
}

func (d *Data) Add(value int) {
	d.Lock() // d.m.Lock()
	defer d.Unlock() // d.m.Unlock()
	
	d.values = append(d.values, value)
}

func main() {
	data := Data{}
	data.Add(100)
}

```

## 5

```go

package main

import (
    "fmt"
    "sync"
)

func WaitToClose(lhs, rhs chan struct{}) {
    lhsClosed, rhsClosed := false, false
    for !lhsClosed || !rhsClosed {
        select {
        case _, ok := <-lhs:
            fmt.Println("lhs", ok)
            if !ok {
                lhsClosed = true
            }
        case _, ok := <-rhs:
            fmt.Println("rhs", ok)
            if !ok {
                rhsClosed = true
            }
        }
    }
}

func main() {
    lhs := make(chan struct{}, 1)
    rhs := make(chan struct{}, 1)

    wg := sync.WaitGroup{}
    wg.Add(1)

    go func() {
        defer wg.Done()
        WaitToClose(lhs, rhs)
    }()

    lhs <- struct{}{}
    rhs <- struct{}{}
    
    close(lhs)
    close(rhs)
    
    wg.Wait()
}

```

```go

package main

import (
    "fmt"
    "sync"
)

func WaitToClose(lhs, rhs chan struct{}) {
    lhsClosed, rhsClosed := false, false
    for !lhsClosed || !rhsClosed {
        select {
        case _, ok := <-lhs:
            fmt.Println("lhs", ok)
            if !ok {
                lhsClosed = true
                lhs = nil // нужно сделать так чтобы при чтении из nil канала была блокировка + мы зануляем только копию канала
            }
        case _, ok := <-rhs:
            fmt.Println("rhs", ok)
            if !ok {
                rhsClosed = true
                rhs = nil
            }
        }
    }
}

func main() {
    lhs := make(chan struct{}, 1)
    rhs := make(chan struct{}, 1)

    wg := sync.WaitGroup{}
    wg.Add(1)

    go func() {
        defer wg.Done()
        WaitToClose(lhs, rhs)
    }()

    lhs <- struct{}{}
    rhs <- struct{}{}
    
    close(lhs) // из-за того что селект выбирает случайно, какую ветку выполнить, а в закрытом канале все еще есть значения, вывод будет неправильный и рандомный
    close(rhs)
    
    wg.Wait()
}

```

## 6

```go

package main

import "fmt"

func main() {
	// если не было бы буфера, то выполнилось бы только ветка default
	ch := make(chan int, 1)
	
	for done := false; !done; {
		select {
		default:
			fmt.Println(3) // т.к ch = nil, то и запись и чтение из nil канала вызывают блокировку
			done = true
		case <- ch:
			fmt.Println(2) // 2 так как default выполняется по умолчанию, а в канал уже записано значение и тк буфер 1, то записать без блокировки уже не могу, а прочитать могу, т.к там уже есть значение
			ch = nil // на самом деле тут и размер буфера не имеет значения, тк канал зануляется и чтение и запись из него без блокировки не сработают
		case ch <- 1:
			fmt.Println(1) // 1 так как default выполняется только если другие ветки не выполняются \ а прочитать из буф канала, если в нем ничего нет без блокировки я не могу, но записать могу
		}
	}
}


```

## Data Race
![[Pasted image 20250831182111.png]]

## Race condition

![[Pasted image 20250831182140.png]]
## 7


```go

package main

import (
    "fmt"
    "sync"
)

func main() {
    text := ""
    wg := sync.WaitGroup{}
    wg.Add(2)

    go func() {
        defer wg.Done()
        text = "hello world" // data race, но если добавить mutex, но останется race condition
    }()

    go func() {
        defer wg.Done()
        fmt.Println(text) // data race
    }()

    wg.Wait()
}
// чтобы починить можно просто убрать всю конкурентность
```

```go

package main

import (
    "fmt"
    "sync"
)

func main() {
    text := ""
    var mutex sync.Mutex
    
    wg := sync.WaitGroup{}
    wg.Add(1)

    go func() {
        defer wg.Done()
        mutex.Lock()
        text = "hello world" // data race, но если добавить mutex, но останется race condition
        mutex.Unlock()
    }()
    
    wg.Add(1)

    go func() {
        defer wg.Done()
        mutex.Lock()
        fmt.Println(text) // data race
        mutex.Unlock()
    }()

    wg.Wait()
}

```


## 8

```go

package main

import "sync"

type Data struct {
    X int
    Y int
}

func main() {
    var data Data
    values := make([]int, 2)

    wg := sync.WaitGroup{}
    wg.Add(2)

    go func() {
        defer wg.Done()
        data.X = 5 
        values[0] = 5
    }()

    go func() {
        defer wg.Done()
        data.Y = 10
        values[1] = 10
    }()
    
    wg.Wait()
}
// data race нету, тк struct и слайс это агрегированные типы данных, тк все равно обращаемся к разным участкам памяти
```