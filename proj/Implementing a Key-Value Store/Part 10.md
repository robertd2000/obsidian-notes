This is Part 10 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts. In this series of articles, I describe the research and process through which I am implementing a key-value database, which I have named “KingDB”. The source code is available at [http://kingdb.org](http://kingdb.org/). Please note that you do not need to read the previous parts to be able to follow what is going on here. The previous parts were mostly exploratory, and starting with [Part 8](http://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/) is perfectly fine.

In this article, I explain the model and inner workings of KingServer, the network server for KingDB. In order to put things into perspective I also cover Nginx, the high-performance HTTP server well-known for its network stack, and how it differs from KingDB.

## TL;DR

If you remember just a few key elements from this article for designing high-performance network programs, they should be the following:

- Design your network protocol with care. When in doubt, copy existing time-tested protocols.
- Minimize the amount of CPU cycles wasted on blocking I/O.
- Keep the resource overhead for each connection as small as possible, so your design can scale sub-linearly with the number of connections.
- Look into the network stack of the Linux kernel and its different tuning parameters. This can lead to enormous and unexpected performance gains.

## 1. Protocol and network clients

The protocol is an important part of any network program: it describes how the server talks to the clients, and poor decisions when designing a protocol can kill even the best architecture. For KingServer, I decided to implement the three basic operations of the Memcached protocol, `get`, `set`, and `delete`, in the exact way they were described in the Memcached repository [[6]](https://codecapsule.com/2016/07/21/implementing-a-key-value-store-part-10-high-performance-networking-kingserver-vs-nginx/#ref).

Using the Memcached protocol has some drawbacks. For example, the current version of KingDB has a multipart API to allow for large objects to be accessed while not requiring too much memory for the clients. However, KingServer is unable to offer that feature over the network as the Memcached protocol does not support multipart access. Indeed, Memcached was never designed to store very large blobs and therefore a multipart API was never needed.

But implementing the Memcached protocol also has advantages. The most important one is that the Memcached protocol has been around for a while and has stood the test of time. Therefore I save myself the labor of designing a new protocol which would inevitably require a lot of work and production testing before all friction points would be ironed out.

Another advantage of implementing the Memcached protocol is that the users of KingServer would be able to use all of the already existing Memcached client librairies in the programming languages they preferred. Thus essentially, I am leveraging the Memcached ecosystem and I save myself from having to implement and support client libraries in various programming languages.

My long-term plan was that if KingDB is to be succesful and used at a larger scale, then it would make sense to gather the most common use cases for using KingDB over the network through KingServer, and then and only then, create a protocol optimized for those use cases.

## 2. Blocking I/O vs non-blocking I/O

In this section I want to briefly compare blocking I/O and non-blocking I/O.

With a blocking I/O approach, calling a blocking syscall would make the caller thread idle and waste CPU cycles until the syscall returns. With a non-blocking I/O approach, the syscall would return right away after it is called, and the caller thread would be notified of incoming I/O operations through an event loop. For that reason, the non-blocking I/O approach is also called asynchronous or event-driven. Between the notification events, the thread is free to perform anything it wants, which means no CPU cycles are lost idling.

The original non-blocking syscalls were select() and poll() and were first introduced in the 1980’s. They become slow under load, due to their inefficient approach for managing many file descriptors. Modern non-blocking applications use epoll() and kqueue() through libraries such as libevent, libev, or libuv.

The rule of thumb is that non-blocking I/O is better, because it avoid wasting CPU cycles by design. However when using a single-threaded non-blocking approach with an event loop, you do have to be careful of what you do with the connection you’re handling at any given iteration of that loop. If you take too long to deal with the chunk of data that this connection needs handled, you run into the risk of making all the other connections assigned to this thread time out.

## Join my email list

## 3. KingServer and the blocking thread-per-connection model

KingServer implements the classic blocking thread-per-connection approach, in which each new connection is handled by a dedicated thread using blocking network I/O. I envisioned using a non-blocking event loop with whatever flavor of epoll() or kqueue() through libev/libuv, which is often regarded as offering better performance as stated in the previous section, but in the end I decided otherwise.

The reason why I went for the thread-per-connection model is simply that KingDB is compressing data and calculating checksums, which can take some time, and I wasn’t entirely sure that this would not cause timeout issues as explained in the previous section. I knew that the thread-per-connection would work, with some known limitations, and as I was looking at getting a first version of KingServer out as quickly as possible, I settled for an inferior yet simpler design first.

For that first version of KingServer, I tried to keep things as simple as possible. KingServer uses a pool of threads created once and for all at start-up, which has the benefit of preventing the overhead of spawning a new thread with every incoming connection. A specialized thread, the receiver thread, listens for incoming connections and dispatches them to a queue.

The available worker threads are monitoring that queue for incoming connections using a C++11 condition_variable. When a connection is picked by a worker, that worker takes care of copying the data from the recv() buffer from kernel space into user space, and does the CPU-intensive tasks of compressing data and calculating checksums.

Having one thread per connection ensures that for example if a connection has to handle a large entry from a client and takes more time, other clients will not timeout as they can be handled by other cores. Figure 10.1 below represents the architecture of KingServer 0.9.0.

KingServer has the following specialized threads:

- **main thread**: keeps the KingServer program, the receiver thread and the worker thread pool running.
- **network receiver**: receives requests from all clients on the network and dispatches them to the worker threads through the queue.
- **worker threads**: each thread in the pool is a worker that picks the next incoming request from the queue, and handles that request by calling the methods of the KingDB object embedded by KingServer. For `put` requests, those threads will also compress data and compute checksums.

![Architecture-of-KingServer](http://codecapsule.com/wp-content/uploads/2016/06/kingserver-architecture.svg)

Figure 10.1: Architecture of KingServer 0.9.0

When a process blocks on an I/O syscall, it loses the opportunity to use the CPU cycles of the core it runs on until the syscall returns. But those cycles are not completely lost, as the OS scheduler will detect the inactivity and will schedule another process to run on that core in the mean time.

Due to the design I chose for KingServer, a lot of CPU time in each thread is spent blocking on network I/O. In order to cope with that, KingServer will by default start with a pool of 150 worker threads, which can be adjusted using a parameter.

At the time I am writing this article, the CPU of a commodity server has an average of 12 or 24 cores. Having 150 worker threads means that there will be more threads than cores in the CPU. One upside of having so many threads is that many connections can be handled simultaneously, as active threads can be scheduled by the OS while other threads are idle and blocking on I/O. However, using so many threads also has a major drawback, which is that CPU cycles will be lost because of lock contention and context switching between threads, also known as thread thrashing.

My choice of the thread-per-connection model was motivated by my desire to release something as quickly as possible, and not by an intent to optimize for performance. The model I have used does the job and supports some level o concurrency, but is definitely not the best. Thus I think it would be only fair and relevant for this article if I covered a network model that allows for even higher networking performance, the Nginx network model.

## 4. Nginx and its non-blocking model

Nginx is an open source high-performance HTTP server that does not use the classic blocking thread-per-connection. Instead, Nginx uses a non-blocking approach, which has proven to be remarkably efficient.

The first way by which Nginx is efficient is that it does not spawn a new process or thread for every incoming connection. On start-up, the Nginx master process forks a pool of worker processes, all of which are single-threaded. Because a single Nginx server can be configured to serve many websites at different address/port pairs, the master Nginx process creates one dedicated socket for each website it needs to serve. All the worker processes share those sockets, on which they listen and accept new requests: they inherit the sockets after the fork() from the master process. This means that all the requests to all the websites handled by a single Nginx server are distributed across all worker processes.

Balancing the requests across workers is left to the OS scheduling mechanism. By default, the `accept_mutex` Nginx parameter is set to `on`, which means that worker processes will accept new connections by turns, i.e. only one worker will accept new connections at a given time. If this parameter is set to `off`, all worker processes will be notified of new connections, which will make some workers waste CPU cycles if the number of connections is low [[1]](https://codecapsule.com/2016/07/21/implementing-a-key-value-store-part-10-high-performance-networking-kingserver-vs-nginx/#ref).

Since Nginx 1.9.1, the `reuseport` parameter makes it possible to use the `REUSE_PORT` socket option, which allows for multiple sockets to listen on the same address/port pair. This enables all the worker threads to have their own dedicated sockets instead of sharing sockets, which reduces lock contention around incoming requests and brings significant performance improvements [[5]](https://codecapsule.com/2016/07/21/implementing-a-key-value-store-part-10-high-performance-networking-kingserver-vs-nginx/#ref).

Each worker has its own run-loop which takes new connections and handles requests from connections already open for that process. The run-loop is non-blocking and event-driven. The master process creates, closes and binds sockets, and the worker processes accept, handle and process requests asynchronously. Figure 10.2 below was taken from the excellent article by Andrew Alexeev about the architecture of Nginx [[2]](https://codecapsule.com/2016/07/21/implementing-a-key-value-store-part-10-high-performance-networking-kingserver-vs-nginx/#ref), and illustrates how the master and worker processes interact.

Memory usage is kept under control because the number of processes and threads is limited by design and the overhead of each additional requests is small. Consequently, Nginx scales sub-linearly with the number of incoming requests. CPU usage is also optimal, as no cycles are wasted on process spawning or thread context switch and the I/O is non-blocking. Keeping a single worker per core allows for good CPU utilization while avoiding thread thrashing and lock contention.

The architecture and internals of Nginx are described in great details in articles from some of the authors themselves [[2, 3, 4]](https://codecapsule.com/2016/07/21/implementing-a-key-value-store-part-10-high-performance-networking-kingserver-vs-nginx/#ref)

![architecture-nginx](https://i0.wp.com/codecapsule.com/wp-content/uploads/2016/06/architecture-nginx.png)

Figure 10.2: Architecture of Nginx, taken from Andrew Alexeev’s article [[2]](https://codecapsule.com/2016/07/21/implementing-a-key-value-store-part-10-high-performance-networking-kingserver-vs-nginx/#ref)

## Conclusion

In this article, I briefly explained the difference between blocking and non-blocking I/O, and I presented two network models, the blocking thread-per-connection model used by KingServer, which I compared to the non-blocking event-driven model of Nginx.

The non-blocking model does have advantages over the blocking one, however one has to be careful with the non-blocking model that the each connection of the event loop are treated as fast as possible to prevent the other already opened connections for this event loop from timing out.

Finally, one thing to keep in mind when designing network programs, but which I have not covered here, is that the Linux network stack is full of parameters and nobs to play with. Spending some time profiling different levels of the network stack and tuning those parameters can lead to enormous and unexpected performance gains, and you should always consider it.

In the next article, I will conclude the series of articles about KingDB, by summarizing what I have achieved, the mistakes I have made, and what I have learned in the process.