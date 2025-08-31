
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