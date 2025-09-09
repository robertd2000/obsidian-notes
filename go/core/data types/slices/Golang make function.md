https://www.zetcode.com/golang/builtins-make/

This tutorial explains how to use the `make` built-in function in Go. We'll cover slice, map, and channel creation with practical examples.

The make function allocates and initializes objects of type slice, map, or channel. Unlike `new`, which returns a pointer, `make` returns an initialized value ready to use.

In Go, `make` is essential for creating complex data structures with specific initial capacities. It ensures proper initialization of internal state.

## Creating a slice with make

The simplest use of `make` creates a slice with specified length and capacity. This example demonstrates basic slice creation.  
**Note:** The capacity parameter is optional.

basic_slice.go

```go


package main

import "fmt"

func main() {
    // Create slice with length 3 and capacity 5
    nums := make([]int, 3, 5)
    
    fmt.Println("Slice:", nums)
    fmt.Println("Length:", len(nums))
    fmt.Println("Capacity:", cap(nums))
    
    // Append elements within capacity
    nums = append(nums, 4, 5)
    fmt.Println("After append:", nums)
    
    // Append beyond capacity
    nums = append(nums, 6)
    fmt.Println("After exceeding capacity:", nums)
}
```


The slice is initialized with zero values for its length. Capacity determines when the underlying array needs reallocation. Appending beyond capacity causes automatic resizing.

## Creating a map with make

We can create maps with initial capacity using `make`. This example shows map creation and basic operations.

```go

package main

import "fmt"

func main() {
    // Create map with initial capacity
    userAges := make(map[string]int, 10)
    
    // Add elements
    userAges["Alice"] = 25
    userAges["Bob"] = 30
    userAges["Charlie"] = 28
    
    fmt.Println("User ages:", userAges)
    
    // Check existence
    if age, exists := userAges["Bob"]; exists {
        fmt.Println("Bob's age:", age)
    }
    
    // Delete element
    delete(userAges, "Charlie")
    fmt.Println("After deletion:", userAges)
}
```
basic_map.go



The capacity hint helps optimize map performance by reducing rehashing. Maps dynamically grow regardless of initial capacity. Elements can be added, checked, and deleted.

## Creating a buffered channel with make

`make` creates channels with specified buffer size. This example shows buffered channel operations.

buffered_channel.go

```go

package main

import "fmt"

func main() {
    // Create buffered channel with capacity 3
    messages := make(chan string, 3)
    
    // Send messages without blocking
    messages <- "first"
    messages <- "second"
    messages <- "third"
    
    // Receive messages
    fmt.Println(<-messages)
    fmt.Println(<-messages)
    fmt.Println(<-messages)
    
    // Close channel
    close(messages)
}

```


Buffered channels allow sending multiple values without blocking. The channel holds values until received. Closing is optional but recommended when done.

## Slice with make and copy

`make` can create slices for copying data. This example demonstrates efficient data copying.

slice_copy.go

```go

package main

import "fmt"

func main() {
    original := []int{1, 2, 3, 4, 5}
    
    // Create slice with exact needed capacity
    copySlice := make([]int, len(original))
    
    // Copy elements
    copied := copy(copySlice, original)
    
    fmt.Println("Original:", original)
    fmt.Println("Copied:", copySlice)
    fmt.Println("Elements copied:", copied)
    
    // Modify copy
    copySlice[0] = 100
    fmt.Println("After modification:")
    fmt.Println("Original:", original)
    fmt.Println("Copied:", copySlice)
}

```


Creating slices with exact capacity avoids unnecessary allocations. The `copy` function returns number of elements copied. Modifying the copy doesn't affect the original.

## Map with make for concurrent access

`make` creates maps that can be used with sync primitives. This example shows thread-safe map access.

concurrent_map.go

```go

package main

import (
    "fmt"
    "sync"
)

type SafeMap struct {
    sync.RWMutex
    data map[string]int
}

func NewSafeMap() *SafeMap {
    return &SafeMap{
        data: make(map[string]int),
    }
}

func (sm *SafeMap) Set(key string, value int) {
    sm.Lock()
    defer sm.Unlock()
    sm.data[key] = value
}

func (sm *SafeMap) Get(key string) (int, bool) {
    sm.RLock()
    defer sm.RUnlock()
    val, exists := sm.data[key]
    return val, exists
}

func main() {
    sm := NewSafeMap()
    
    var wg sync.WaitGroup
    
    // Concurrent writes
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            key := fmt.Sprintf("key%d", i)
            sm.Set(key, i)
        }(i)
    }
    
    wg.Wait()
    
    // Read values
    for i := 0; i < 10; i++ {
        key := fmt.Sprintf("key%d", i)
        if val, exists := sm.Get(key); exists {
            fmt.Printf("%s: %d\n", key, val)
        }
    }
}

```



The `SafeMap` wrapper provides thread-safe access to a map created with `make`. Mutexes protect concurrent access. This pattern is common in concurrent Go programs.

## Source

[Go language specification](https://go.dev/ref/spec#Make)

This tutorial covered the `make` function in Go with practical examples of slice, map, and channel creation.