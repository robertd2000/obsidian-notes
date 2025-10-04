From [the docs](https://golang.org/pkg/sync/#RWMutex) (emphasis mine):

> A RWMutex is a reader/writer mutual exclusion lock. _The lock can be held by an arbitrary number of readers or a single writer._ The zero value for a RWMutex is an unlocked mutex.

In other words, readers don't have to wait for each other. They only have to wait for writers that are holding the lock.

A sync.RWMutex is thus preferable for data that is mostly read, and the resource that is saved compared to a sync.Mutex is time.