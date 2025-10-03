## Introduction

Pooling is a resource management technique that involves creating and maintaining a pool of reusable resource instances in advance, allowing for quick allocation and deallocation of these resources when needed. Goroutines are lightweight "threads" in the Go programming language, but a large number of goroutines can still consume significant resources. A Worker Pool, using pooling techniques, can maintain a certain number of goroutines and only let them execute tasks, reducing resource waste.

This article will introduce one implementation approach for a Worker Pool and analyze the source code of a popular Worker Pool library, [gammazero/workerpool](https://github.com/gammazero/workerpool). Finally, we will implement a similar Worker Pool called [VIOLIN](https://github.com/B1NARY-GR0UP/violin) to gain a better understanding of Worker Pools.

## [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#how-to-write-a-worker-pool)How to Write a Worker Pool?

Based on the brief introduction of the Worker Pool in the previous section, we learned that implementing a Worker Pool involves maintaining a certain number of goroutines and letting them handle the submitted tasks/jobs. The approach for writing a Worker Pool becomes straightforward:

- Set a specific limit on the number of goroutines in the Worker Pool. The number of goroutines maintained by the Worker Pool should not exceed this limit.
- When a task is submitted, allocate a goroutine from the Worker Pool to execute it.
- If there is an idle worker (goroutine), assign the task to the idle worker. Otherwise, check if the limit of goroutines has been reached.
- If the limit is reached, cache the task in a queue (or use another handling mechanism). If the limit has not been reached, create a new worker to execute the task.

For a more detailed understanding of the logic, please refer to the diagram below. The explanation of when to consume tasks from the task queue will be discussed in the code analysis section.

[![Image description](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwpf84b4nn567dgzme84y.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwpf84b4nn567dgzme84y.png)

## [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#code-analysis)Code Analysis

Now that we have a general understanding of how a Worker Pool works, we can start writing the code. Here, we'll first examine an excellent open-source implementation of a Worker Pool with 1.1k stars on GitHub. The project can be found [here](https://github.com/gammazero/workerpool).

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#main-struct-workerpool)Main struct WorkerPool

Let's take a look at the main structure of this Worker Pool:  

```
type WorkerPool struct {
    maxWorkers   int
    taskQueue    chan func()
    workerQueue  chan func()
    stoppedChan  chan struct{}
    stopSignal   chan struct{}
    waitingQueue deque.Deque[func()]
    stopLock     sync.Mutex
    stopOnce     sync.Once
    stopped      bool
    waiting      int32
    wait         bool
}
```

We will focus on the following properties:

- **maxWorkers**: The maximum number of goroutines (i.e., workers) maintained by the Worker Pool.
- **taskQueue**: The task queue where tasks are submitted and stored.
- **workerQueue**: The worker queue from which workers retrieve tasks to execute.
- **waitingQueue**: The waiting queue, used to cache tasks when there are no available workers and the current number of workers has reached the maximum limit.

Some important points to note:

- The specific data type of elements stored in all three queues is `func()`, representing the type of submitted tasks (consider it as a function).
- **Here, two queues, `taskQueue` and `workerQueue`, are used. When a task is submitted, it is first added to the `taskQueue`. Then, a task is retrieved from the `taskQueue` and passed to the `workerQueue` for execution by a worker. It is not directly consumed by the workers from the `taskQueue`.**
- The data type of `waitingQueue` is a custom data structure implemented by the author of the workerpool library. You can think of it as a general FIFO queue. When we implement our own Worker Pool later, we can replace it with existing data structures like `chan`, `list.List`, etc.

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#new)New

Next, let's take a look at the `New` function, which is the function for creating a `WorkerPool` object.  

```
func New(maxWorkers int) *WorkerPool {
    // There must be at least one worker.
    if maxWorkers < 1 {
        maxWorkers = 1
    }

    pool := &WorkerPool{
        maxWorkers:  maxWorkers,
        taskQueue:   make(chan func()),
        workerQueue: make(chan func()),
        stopSignal:  make(chan struct{}),
        stoppedChan: make(chan struct{}),
    }

    // Start the task dispatcher.
    go pool.dispatch()

    return pool
}
```

As we can see, the overall structure is quite clear. The initial condition ensures that the WorkerPool maintains at least one goroutine. Then, the WorkerPool object is initialized. It's worth noting that both the `taskQueue` and `workerQueue` attributes are unbuffered channels. After that, a goroutine is started to execute the core method of the WorkerPool, `pool.dispatch`. Finally, the WorkerPool object is returned.

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#submit)Submit

Before we dive into the core method `pool.dispatch`, let's take a look at a few simple methods to boost confidence.  

```
func (p *WorkerPool) Submit(task func()) {
    if task != nil {
        p.taskQueue <- task
    }
}
```

The `Submit` method is used to submit tasks and can be called by directly checking if the task is not `nil` and then passing it to the `taskQueue`. By the way, let's briefly discuss what happens if we pass `nil` to the channel. Although the channel can receive `nil` without any issues, attempting to run it will result in a null pointer exception.  

```
func main() {
    ch := make(chan func())
    go func() {
        ch <- nil
    }()
    res, ok := <-ch
    fmt.Println(res, ok) // <nil> true
    res()                // panic: runtime error: invalid memory address or nil pointer dereference
}
```

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#submitwait)SubmitWait

There is a twin method to `Submit`, which blocks and waits for the submitted task to complete execution.  

```
func (p *WorkerPool) SubmitWait(task func()) {
    if task == nil {
        return
    }
    doneChan := make(chan struct{})
    p.taskQueue <- func() {
        task()
        close(doneChan)
    }
    <-doneChan
}
```

The specific implementation involves wrapping the submitted task once and then using a channel to accomplish this, which is quite clever.

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#dispatch)dispatch

The core method of the WorkerPool encompasses the complete workflow of the Worker Pool that we analyzed earlier.

This is a very lengthy method. To facilitate explanation and understanding, I will first provide the complete method, and then I will explain each code segment one by one. Readers can either clone the project and refer to the code while reading, or they can directly follow the code analysis below.  

```
func (p *WorkerPool) dispatch() {
    defer close(p.stoppedChan)
    timeout := time.NewTimer(idleTimeout)
    var workerCount int
    var idle bool
    var wg sync.WaitGroup

Loop:
    for {
        // As long as tasks are in the waiting queue, incoming tasks are put
        // into the waiting queue and tasks to run are taken from the waiting
        // queue. Once the waiting queue is empty, then go back to submitting
        // incoming tasks directly to available workers.
        if p.waitingQueue.Len() != 0 {
            if !p.processWaitingQueue() {
                break Loop
            }
            continue
        }

        select {
        case task, ok := <-p.taskQueue:
            if !ok {
                break Loop
            }
            // Got a task to do.
            select {
            case p.workerQueue <- task:
            default:
                // Create a new worker, if not at max.
                if workerCount < p.maxWorkers {
                    wg.Add(1)
                    go worker(task, p.workerQueue, &wg)
                    workerCount++
                } else {
                    // Enqueue task to be executed by next available worker.
                    p.waitingQueue.PushBack(task)
                    atomic.StoreInt32(&p.waiting, int32(p.waitingQueue.Len()))
                }
            }
            idle = false
        case <-timeout.C:
            // Timed out waiting for work to arrive. Kill a ready worker if
            // pool has been idle for a whole timeout.
            if idle && workerCount > 0 {
                if p.killIdleWorker() {
                    workerCount--
                }
            }
            idle = true
            timeout.Reset(idleTimeout)
        }
    }

    // If instructed to wait, then run tasks that are already queued.
    if p.wait {
        p.runQueuedTasks()
    }

    // Stop all remaining workers as they become ready.
    for workerCount > 0 {
        p.workerQueue <- nil
        workerCount--
    }
    wg.Wait()

    timeout.Stop()
}
```

In fact, when we look at the entire method, it is still very clear, and the author has thoughtfully added comments at the main logic branches.

- Let's start by examining the variable declaration section at the beginning of this method:

```
  defer close(p.stoppedChan)
  timeout := time.NewTimer(idleTimeout)
  var workerCount int
  var idle bool
  var wg sync.WaitGroup
```

- **timeout**: The WorkerPool sets a timeout for the idle workers, which represents the maximum time a worker can remain idle before it's considered expired. This prevents idle workers from consuming system resources indefinitely. In this case, the timeout is statically declared as 2 seconds (`2 * time.Second`).
- **workerCount**: This variable is used to keep track of the current number of running workers. It will be compared with the maximum allowed number of workers configured for the pool.
- **idle**: This is a flag to mark if any worker has timed out. It is initially set to `false` and is set to `true` if any worker times out. It will be set back to `false` after creating a new worker or adding a task to the task queue.
- **wg**: This is a WaitGroup used to keep track of the number of active workers. It is incremented by 1 each time a new worker is created and blocks until all workers have finished their execution before proceeding to worker destruction.
    
    - The next part of the method is a massive `for` loop with nested `select` statements. Let's skip the logic related to `waitingQueue` for now and focus on the main `select` section:

```
  LOOP:
    for {
          // ... 

          select {
          case task, ok := <-p.taskQueue:
              if !ok {
                  break Loop
              }
              // Got a task to do.
              select {
              case p.workerQueue <- task:
              default:
                  // Create a new worker, if not at max.
                  if workerCount < p.maxWorkers {
                      wg.Add(1)
                      go worker(task, p.workerQueue, &wg)
                      workerCount++
                  } else {
                      // Enqueue task to be executed by next available worker.
                      p.waitingQueue.PushBack(task)
                      atomic.StoreInt32(&p.waiting, int32(p.waitingQueue.Len()))
                  }
              }
              idle = false
          case <-timeout.C:
              // Timed out waiting for work to arrive. Kill a ready worker if
              // pool has been idle for a whole timeout.
              if idle && workerCount > 0 {
                  if p.killIdleWorker() {
                      workerCount--
                  }
              }
              idle = true
              timeout.Reset(idleTimeout)
          }
      }
  // ...
```

- The first `select` statement within the `for` loop has two `case` options: one for receiving a task and the other for worker timeout (no new task received within two seconds).
    
- Let's start with the case for receiving a task. First, it checks if an actual task is received. If `ok` is `false`, it means that the `taskQueue` channel has been closed. In this case, the loop is exited. The reason for using a label is that within the nested `select` inside the for-select, a regular `break` statement would only exit the current `select` block and continue executing the code outside the `select`, including the next iteration of the `for` loop. To exit the outer `for` loop directly, a label is used.
    
    Then, another `select` statement is used. If there is an idle worker, the task is assigned to that worker (`p.workerQueue <- task`). If there are no idle workers, it checks if the current number of running workers has reached the maximum limit. If not, a new goroutine (worker) is started, and the `workerCount` and `sync.WaitGroup` are incremented by 1. Since the `dispatch` method does not have concurrency issues, it can be directly incremented. If the maximum number of workers has been reached, the current task is cached in the queue, and the number of waiting tasks is recorded. Since `p.waiting` is a shared resource, atomic operations are used to ensure concurrent safety.
    
- The case for worker timeout first checks if `idle` is `true` and the number of workers is greater than 0. Therefore, during the first timeout, a worker is not killed. Instead, `idle` is set to `true`, and the timer is reset. It's worth noting that the `p.killIdleWorker` method may not necessarily kill a worker; it attempts to do so and returns the result. If a worker is successfully killed, the worker count is decremented by one.
    
    - Now let's look at the part we skipped earlier, which is the beginning of the `for` loop:

```
  if p.waitingQueue.Len() != 0 {
      if !p.processWaitingQueue() {
          break Loop
      }
      continue
  }
```

This section is responsible for consuming the cached tasks. The `p.processWaitingQueue` method either consumes a cached task or caches an incoming task. Similarly, it checks if the `taskQueue` has been closed, and if so, it directly breaks the loop and exits.

- The final part is the cleanup code outside the `for` loop:

```
  // If instructed to wait, then run tasks that are already queued.
  if p.wait {
      p.runQueuedTasks()
  }

  // Stop all remaining workers as they become ready.
  for workerCount > 0 {
      p.workerQueue <- nil
      workerCount--
  }
  wg.Wait()

  timeout.Stop()
```

The `p.wait` flag is used as a marker during the `Stop` operation, but we can skip it for now. The following code is responsible for stopping all workers and stopping the timer.

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#worker)worker

Yes, the WorkerPool creates a new worker by calling `go worker(task, p.workerQueue, &wg)`, passing the task to be executed, the workerQueue, and the WaitGroup as parameters. It cleverly executes the task if it is not `nil` and then blocks to receive a new task until a `nil` task is received, which ends the loop. The WaitGroup is decremented by one, signaling that the worker has finished its execution. It's a clever implementation, and I wouldn't have thought of it myself either!  

```
// worker executes tasks and stops when it receives a nil task.
func worker(task func(), workerQueue chan func(), wg *sync.WaitGroup) {
    for task != nil {
        task()
        task = <-workerQueue
    }
    wg.Done()
}
```

That concludes the analysis of the core source code of the WorkerPool. Now, you can try implementing your own Worker Pool.

## [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#implemented-by-myself)Implemented by myself

[VIOLIN](https://github.com/B1NARY-GR0UP/violin) is a Worker Pool I wrote, inspired by [gammazero/workerpool](https://github.com/gammazero/workerpool). It provides a richer API and options.

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#workernum)WorkerNum

`WorkerNum` is an interface provided by VIOLIN to retrieve the current number of running workers. Instead of using `workerCount` as a local variable in the `dispatch` method, VIOLIN maintains the worker count as a property of the main structure. However, to ensure concurrency safety, atomic operations are used, which may result in performance overhead.  

```
type Violin struct {
    options *options

    mu   sync.Mutex
    once sync.Once

    workerNum      uint32
    taskNum        uint32
    waitingTaskNum uint32
    status         uint32

    workerChan   chan func()
    taskChan     chan func()
    waitingChan  chan func()
    dismissChan  chan struct{}
    pauseChan    chan struct{}
    shutdownChan chan struct{}
}
```

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#functional-options-pattern)Functional Options Pattern

Functional Options is a programming pattern that I personally enjoy using. VIOLIN adopts the Functional Options pattern to provide three configuration options with corresponding default values, as shown in the table below:

|Option|Default|Description|
|---|---|---|
|`WithMaxWorkers`|`5`|Set the maximum number of workers|
|`WithWaitingQueueSize`|`64`|Set the size of the waiting queue|
|`WithWorkerIdleTimeout`|`time.Second * 3`|Set the destroyed timeout of idle workers|

- As compared to the fixed 2-second `idleTimeout` in workerpool, VIOLIN allows users to configure it according to their needs.
- Additionally, VIOLIN replaces the task cache queue's data structure with channels, so users need to set the channel size or use the default value. In the future, it may switch back to a queue implementation, removing the limitation on the cache queue's size.

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#status)Status

VIOLIN maintains the `status` field in the main structure to facilitate users and internal components in accessing the current state of VIOLIN. There are four possible states:  

```
const (
    _ uint32 = iota
    statusInitialized
    statusPlaying
    statusCleaning
    statusShutdown
)
```

- **statusInitialized**: Initialized state
- **statusPlaying**: VIOLIN is running normally
- **statusCleaning**: After calling `ShutdownWait`, VIOLIN is waiting for the remaining tasks in the task cache queue to complete.
- **statusShutdown**: VIOLIN has been shut down.

Users can determine the state of VIOLIN by using the provided methods: `IsPlaying`, `IsCleaning`, and `IsShutdown`.

### [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#problem-encountered)Problem Encountered

The following is a problem I encountered while writing VIOLIN. Many of these issues arose from overcomplicating the logic and getting lost in my own thoughts, leading to various concurrency and workflow problems. Examples include "send on closed channel" errors and data races.  

```
if int(v.WorkerNum()) < v.MaxWorkerNum() {
    _ = atomic.AddUint32(&v.workerNum, 1)
    go v.recruit(wg)
}
```

This is the logic for creating a new worker when the current number of running workers is less than the maximum set value. In the initial version of the code, the code that incrementally increases `workerNum` through atomic operations was placed inside the `recruit` function. This resulted in the actual number of running workers exceeding the maximum value by several multiples at times. After searching for a while, I realized that the issue was due to the execution order.

The solution is to **increment the value immediately after the conditional check**. Otherwise, because the loop executes faster than starting a goroutine and incrementing the value atomically inside the `recruit` function, multiple goroutines can be created for the same conditional check.

## [](https://dev.to/justlorain/go-how-to-write-a-worker-pool-1h3b#conclusion)Conclusion

That concludes all the content of this article. We started with the approach to building a Worker Pool, then examined an open-source implementation of a Worker Pool, and finally implemented our own Worker Pool. I hope this has been helpful in your understanding and use of Worker Pools. If there are any mistakes or questions, please feel free to comment or reach out via private message.