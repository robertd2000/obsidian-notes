Concurrency is a fundamental aspect of modern Go programming, enabling efficient utilization of system resources and improved application performance. However, managing concurrent access to shared resources can be challenging, leading to race conditions and data corruption. Mutexes and RWMutexes are essential synchronization primitives in Go that provide mechanisms to protect shared data and ensure data integrity in concurrent environments. Understanding these primitives in depth is crucial for building robust and reliable concurrent applications.

## Understanding Mutexes

A mutex, short for "mutual exclusion," is a synchronization primitive that allows only one goroutine to access a shared resource at a time. This prevents race conditions and ensures data consistency when multiple goroutines are operating on the same data.

### Basic Mutex Operations: Lock and Unlock

The `sync.Mutex` type in Go provides two primary methods: `Lock()` and `Unlock()`.

- `Lock()`: Acquires the mutex. If the mutex is already locked by another goroutine, the calling goroutine will block until the mutex becomes available.
- `Unlock()`: Releases the mutex. It's crucial to ensure that the mutex is unlocked only by the same goroutine that locked it.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	counter int
	mutex   sync.Mutex
)

func increment() {
	mutex.Lock() // Acquire the lock
	defer mutex.Unlock() // Ensure the lock is released when the function exits

	// Simulate some work
	time.Sleep(time.Millisecond)
	counter++
	fmt.Printf("Counter: %d\n", counter)
}

func main() {
	var wg sync.WaitGroup
	numGoroutines := 5

	wg.Add(numGoroutines)
	for i := 0; i < numGoroutines; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100; j++ {
				increment()
			}
		}()
	}

	wg.Wait()
	fmt.Println("Final Counter:", counter)
}
```

In this example:

- A global `counter` variable is shared among multiple goroutines.
- A `sync.Mutex` is used to protect the `counter` from race conditions.
- The `increment()` function acquires the mutex before accessing the `counter` and releases it afterward using `defer`.
- Multiple goroutines call the `increment()` function concurrently.
- The `defer mutex.Unlock()` ensures that the mutex is always released, even if a panic occurs within the critical section.

### Common Pitfalls with Mutexes

- **Forgetting to Unlock:** Failing to unlock a mutex can lead to deadlocks, where goroutines are blocked indefinitely waiting for a mutex that will never be released. Using `defer mutex.Unlock()` is a good practice to avoid this.
- **Unlocking an Unlocked Mutex:** Attempting to unlock a mutex that is not currently locked will result in a runtime error.
- **Deadlocks:** Deadlocks can occur when multiple goroutines are waiting for each other to release mutexes, creating a circular dependency. Careful design and ordering of mutex acquisitions are essential to prevent deadlocks.

### Advanced Mutex Usage: TryLock

The `TryLock()` method attempts to acquire the mutex without blocking. It returns `true` if the mutex was successfully acquired and `false` otherwise. This can be useful in situations where you want to avoid blocking indefinitely.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	resourceBusy bool
	mutex      sync.Mutex
)

func accessResource(id int) {
	if mutex.TryLock() {
		defer mutex.Unlock()
		resourceBusy = true
		fmt.Printf("Goroutine %d: Acquired lock, accessing resource\n", id)
		time.Sleep(2 * time.Second) // Simulate resource access
		resourceBusy = false
		fmt.Printf("Goroutine %d: Released lock\n", id)
	} else {
		fmt.Printf("Goroutine %d: Could not acquire lock, resource busy\n", id)
	}
}

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			accessResource(id)
		}(i)
		time.Sleep(time.Second) // Stagger goroutine start times
	}

	wg.Wait()
	fmt.Println("Done")
}
```

In this example:

- `TryLock()` is used to attempt to acquire the mutex.
- If the mutex is acquired successfully, the goroutine accesses the resource and then releases the mutex.
- If the mutex cannot be acquired, the goroutine reports that the resource is busy.

## Understanding RWMutexes

An RWMutex (Read-Write Mutex) is a more specialized type of mutex that allows multiple readers to access a shared resource concurrently, but only allows one writer at a time. This can improve performance in scenarios where reads are much more frequent than writes.

### RWMutex Operations: RLock, RUnlock, Lock, and Unlock

The `sync.RWMutex` type provides four primary methods:

- `RLock()`: Acquires a read lock. Multiple goroutines can hold read locks simultaneously.
- `RUnlock()`: Releases a read lock.
- `Lock()`: Acquires a write lock. This blocks all other goroutines (both readers and writers) until the write lock is released.
- `Unlock()`: Releases a write lock.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	data      map[string]string
	rwMutex sync.RWMutex
)

func readData(key string, id int) {
	rwMutex.RLock() // Acquire read lock
	defer rwMutex.RUnlock() // Release read lock

	fmt.Printf("Reader %d: Reading key %s\n", id, key)
	time.Sleep(time.Millisecond * 100) // Simulate reading
	value := data[key]
	fmt.Printf("Reader %d: Key %s has value %s\n", id, key, value)
}

func writeData(key, value string, id int) {
	rwMutex.Lock() // Acquire write lock
	defer rwMutex.Unlock() // Release write lock

	fmt.Printf("Writer %d: Writing key %s with value %s\n", id, key, value)
	time.Sleep(time.Millisecond * 200) // Simulate writing
	data[key] = value
	fmt.Printf("Writer %d: Finished writing key %s\n", id, key)
}

func main() {
	data = make(map[string]string)
	var wg sync.WaitGroup

	// Start multiple readers
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			readData("mykey", id)
		}(i)
	}

	// Start a writer
	wg.Add(1)
	go func() {
		defer wg.Done()
		writeData("mykey", "myvalue", 1)
	}()

	// Start more readers after the writer
	for i := 4; i <= 5; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			readData("mykey", id)
		}(i)
	}

	wg.Wait()
	fmt.Println("Done")
}
```

In this example:

- A `sync.RWMutex` is used to protect a shared `data` map.
- Multiple readers can access the map concurrently using `RLock()` and `RUnlock()`.
- A writer acquires an exclusive lock using `Lock()` and `Unlock()` to modify the map.

### Use Cases for RWMutex

RWMutexes are particularly useful in scenarios where:

- Reads are much more frequent than writes.
- The cost of acquiring a write lock is high.
- Data consistency is critical.

Examples include:

- Caching systems: Multiple clients can read from the cache concurrently, while only one client can update the cache at a time.
- Configuration management: Multiple services can read configuration data, while only one service can update the configuration.
- Game servers: Multiple players can read game state, while only the game logic can update the game state.

### Potential Issues with RWMutex

- **Writer Starvation:** If there are many readers, a writer might be blocked indefinitely waiting for all readers to release their read locks. This can be mitigated by using techniques like priority queues or by limiting the number of concurrent readers.
- **Complexity:** RWMutexes add complexity to the code, as you need to carefully manage both read and write locks.

## Choosing Between Mutex and RWMutex

The choice between `sync.Mutex` and `sync.RWMutex` depends on the specific access patterns of the shared resource.

- Use `sync.Mutex` when:
    - The resource is frequently written to.
    - The read and write operations are equally frequent.
    - Simplicity is a priority.
- Use `sync.RWMutex` when:
    - The resource is read much more frequently than it is written to.
    - Performance is critical.
    - The added complexity is acceptable.

## Practical Examples

### Example 1: Concurrent Counter with RWMutex

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type SafeCounter struct {
	mu    sync.RWMutex
	value int
}

func (c *SafeCounter) Inc() {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.value++
}

func (c *SafeCounter) Value() int {
	c.mu.RLock()
	defer c.mu.RUnlock()
	return c.value
}

func main() {
	counter := SafeCounter{value: 0}

	var wg sync.WaitGroup
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			counter.Inc()
		}()
	}

	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			time.Sleep(time.Millisecond * 10)
			fmt.Println(counter.Value())
		}()
	}

	wg.Wait()
	fmt.Println("Final Counter Value:", counter.Value())
}
```

In this example, the `SafeCounter` struct uses an `RWMutex` to protect the `value` field. The `Inc()` method acquires a write lock to increment the counter, while the `Value()` method acquires a read lock to retrieve the current value.

### Example 2: Concurrent Map with Mutex

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type ConcurrentMap struct {
	mu    sync.Mutex
	items map[string]int
}

func (cm *ConcurrentMap) Set(key string, value int) {
	cm.mu.Lock()
	defer cm.mu.Unlock()
	cm.items[key] = value
}

func (cm *ConcurrentMap) Get(key string) (int, bool) {
	cm.mu.Lock()
	defer cm.mu.Unlock()
	val, ok := cm.items[key]
	return val, ok
}

func (cm *ConcurrentMap) Delete(key string) {
	cm.mu.Lock()
	defer cm.mu.Unlock()
	delete(cm.items, key)
}

func main() {
	cmap := ConcurrentMap{items: make(map[string]int)}

	var wg sync.WaitGroup
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func(key string, value int) {
			defer wg.Done()
			cmap.Set(key, value)
			time.Sleep(time.Millisecond * 10)
			val, ok := cmap.Get(key)
			if ok {
				fmt.Printf("Key: %s, Value: %d\n", key, val)
			}
		}(fmt.Sprintf("key-%d", i), i*10)
	}

	wg.Wait()
	fmt.Println("Done")
}
```

In this example, the `ConcurrentMap` struct uses a `Mutex` to protect the `items` map. The `Set()`, `Get()`, and `Delete()` methods all acquire the mutex before accessing the map.

## Exercises

1. **Implement a thread-safe stack:** Create a stack data structure that can be safely accessed by multiple goroutines concurrently. Use a mutex to protect the stack's internal data. Implement `Push()` and `Pop()` methods.
2. **Implement a rate limiter:** Design a rate limiter that allows a certain number of requests per second. Use a mutex to protect the rate limiter's internal state. Implement an `Allow()` method that returns `true` if a request is allowed and `false` otherwise.
3. **Modify the concurrent map example to use RWMutex:** Analyze the access patterns of the concurrent map and determine if using an `RWMutex` would improve performance. Modify the code to use an `RWMutex` and compare the performance with the original implementation.

## Summary and Next Steps

In this lesson, we explored the concepts of mutexes and RWMutexes in Go, which are essential for managing concurrent access to shared resources and preventing race conditions. We learned how to use the `sync.Mutex` and `sync.RWMutex` types, and we discussed common pitfalls and best practices.

In the next lesson, we will delve into implementing worker pools for task distribution, which is another important concurrency pattern in Go. We will learn how to use goroutines and channels to create worker pools that can efficiently process a large number of tasks concurrently.