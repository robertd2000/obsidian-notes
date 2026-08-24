This is Part 8 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts. In this series of articles, I describe the research and process through which I am implementing a key-value database, which I have named “KingDB”. The source code is available at [http://kingdb.org](http://kingdb.org/). Please note that you do not need to read the previous parts to be able to follow. The previous parts were mostly exploratory, and starting with Part 8 is perfectly fine.

In the previous articles, I have laid out the research and discussion around what needs to be considered when implementing a new key-value store. In this article, I will present the architecture of KingDB, the key-value store of this article series that I have finally finished implementing.

## 1. Introduction

KingDB is a persistent key-value store. All data is written to disk to minimize data loss in case of system failure. Each entry that KingDB stores is a pair of the type <key, value>, where key and value are byte arrays. Keys can have a size up to (2^31 – 1) bytes, and values a size of up to 2^64 bytes.

KingDB can be used as an embedded database in another program, either by compiling the KingDB source code with the program, or by compiling KingDB as a library and linking to it. KingDB can also be used as a network database through KingServer, that embeds KingDB and exposes a network interface which network clients can access through the Memcached protocol [[1]](https://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/#ref). Therefore, KingDB can be used through a network simply by using any of the already available Memcached libraries in any programming language. Note that currently, only a subset of the Memcached protocol is supported: the get, set, and delete commands.

Data is persisted to disk using log-structured storage, in which all writes and deletes are written to disk sequentially like in a log file. It is similar to Write-Ahead Logging (WAL), except that unlike with WAL where the logs have to be merged into the database at some point, log-structure storage only needs to compact old unused entries. The data format used by KingDB is the Hashed String Table, or HSTable. Each HSTable is just a flat file that contains entries, and at the end of that file, an Offset Array that contains the hashed keys and location of the entries in the file for rapid indexing.

I have used LevelDB as a model for KingDB, which is why if you are familiar with LevelDB, you will find similarities in the way that the code is organized in KingDB. LevelDB is a little marvel, and drawing inspiration from it has been for me a great way to learn and improve. If you haven’t had a chance to go through the source code of LevelDB, I recommend that you do so, it’s among the best source code I have ever read! The architecture and source code of LevelDB are just very clear and explicit. In addition, the authors of LevelDB have implemented in a concise manner all the components required for a medium-sized C++ project: error management, logging, unit-testing framework, benchmarking, etc., which is great if you want to avoid external dependencies and learn how to implement a minimal self-sustaining system. However the resemblance between KingDB and LevelDB stops there: all the internal algorithms and data structures in KingDB are new, and I implemented them from scratch.

## 2. Database, Snapshots and Iterators

KingDB’s default interface is the Database class, which offers three basic operations: Get(), Put(), and Delete().

KingDB also offers read-only iterators that can iterate over all the entries stored in a database. Because KingDB uses a hash table to store the locations of entries, the iterators will not return entries in order of keys, but unordered, in the sequence with which they have been written to disk.

Another way to access a KingDB database is to do it through a read-only snapshot. The Snapshot class allows all the concurrent random operations that the regular Database class does, and it guarantees a consistent view of the data.

## 3. Architectural overview

Figure 8.1 below is an architectural overview of KingDB. All layers are represented, from the client application at the top, down to the storage engine and file system at the bottom. Step-by-step explanations of the read and write processes are also represented on the diagram using numbered circles. They are enough to get a good understanding as to how the data is flowing, and how the main components and threads are interacting inside KingDB. In the remaining sections of this article, I give more details regarding the architectural choices and the role of all the components in this architecture.

![Architecture-of-KingDB-web](http://codecapsule.com/wp-content/uploads/2015/05/Architecture-of-KingDB-vecto.svg)

Figure 8.1: Architecture of KingDB v0.9.0

## 4. Index

The role of the index is to quickly find the location of an entry in the secondary storage using only the key of that entry. For this, KingDB uses a pair of 64-bit integers for each entry: the hashed key and the location:

- The **hashed key** of an entry is a 64-bit integer computed by passing the key of that entry through a selected hash function. Because hashing collisions exist, entries with different keys can have the same hashed key.
- The **location** of an entry is a unique 64-bit integer: the first 32 bits are pointing to the HSTable, i.e. path of the file on disk, and the last 32 bits are pointing to the offset in the HSTable file where the entry can be found.

KingDB uses a std::multimap as index, and for each hashed key associates all the locations where entries having that hashed key can be found. When the location of a new entry is stored, the hashed key for that entry is used as the key for the std::multimap, and the location is inserted in the bucket for that hashed key. KingDB adds locations as they are incoming, thus if an entry has multiple versions, i.e. a single entry was written multiple times, each time with the same key but with a different value, then the latest version will be added last to the std::multimap. When the location of an entry is retrieved, its hashed key is computed and used to find the list of locations where entries sharing that hashed key can be found. The locations are then considered in reverse order, which guarantees that the latest version of the retrieved entry will always be found first, and the older versions are simply ignored [[2]](https://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/#ref). This is also one of the reasons why the writes are fast: in case an entry has multiple versions, all versions are stored in the index and on disk. Same goes with the deletes: nothing is really deleted, KingDB only registers the deletion orders and hides the deleted entries from the user. The compaction process later takes care, in the background, of removing old versions and deleted entries from the file storage and from the index.

Accesses to the index are done in the form of multiple readers, single writer, using a single lock to ensure that the index is not modified while it is being read. Because all updates to the index are happening through the Storage Engine, they are grouped and applied by small batches within short time intervals, which minimizes the delay during which the index needs to be locked and any chance of blocking the reads while the index is updated.

Using a std::multimap has been one of the most important design decisions that I took early on for KingDB. I considered implementing my own lock-free version of Robin Hood hashing and use it for the index of KingDB, but it really felt like premature optimization. So I decided that for the first version of KingDB, I would use a container from the standard library and see how far it could take me. I am happy that I used std::multimap because it saved time and enabled me to focus on the on-disk data format, however it did introduce some drawbacks:

1. Data is hashed twice: the first time when the _hashed key_ is computed, and the second time when the std::multimap computes another hash from the hashed key, for its own internal representation. Hashing data is a CPU-costly operation.
2. The entire index needs to be stored in memory at all time. This is fine if the database is small, but this can be several GBs of data if the database grows.
3. This makes snapshots costly. Indeed, snapshots require their own index to guarantee that they can offer a read-only consistent view of a database. So each time a snapshot is created, the entire index has to be copied and stored in an extra memory space. For large databases, this means several GBs of data duplicated in the memory.

It is clear that the only way to improve the design and the overall database performance at this stage is to implement a custom data structure that will serve as the index for KingDB. Ideally, this index will be stored on disk so it won’t take too much memory while it could still be loaded using read-only memory maps, and another good feature for this index would be to be in multiple parts, so that snapshots wouldn’t need to duplicate the entire index and could only lock the parts of the index that they need.

## Join my email list

## 5. The ByteArray class: RAII and zero-copy buffering through memory maps

All reads to entries are made through the ByteArray class, which provides a pointer to an array and a size. The ByteArray class encapsulates how the entry is really accessed, and in the current version of KingDB, it is done through read-only memory maps. Resources such as file descriptors and memory map pointers are held by the ByteArray class, and released by its destructor when they are no longer needed. Thus ByteArray is doing RAII, Resource Acquisition Is Initialization. Other sub-classes of ByteArray also allow for entries to be stored in memory, or for memory locations that are shared among multiple ByteArray instances, through a shared_ptr.

The reason why I have chosen to use memory maps to serve reads is because they allow for zero-copy buffering. Indeed, with the memory map, when data is read from disk to be sent on the network, it is first read from disk into a buffer in kernel space, and then it stays in kernel space when it is used by send(). This prevents a useless copy of the data into a buffer of the user space. Copying buffers around, all the more so between user and kernel space, is time consuming, thus this trick allows KingDB to save time and serve data faster. Another interesting point is that the data retrieved is likely to be larger than the maximum payload that can transit on a network, thus any type of caching while reading data from disk is going to increase the throughput. Memory maps are an easy way to get such caching, by reading only what is needed from the buffer and letting the kernel handle the caching.

## 6. Multipart API

During the development of KingDB, I wanted to split the development of the storage engine and of the network server. My plan was to first work on the core storage engine and get it to an acceptable stage with enough of the bugs washed out, and then work on the network part. My expectation was that if a bug occurred, I would know more easily if it came from the storage engine or the network. But this has bitten me pretty hard afterwards: I had design KingDB to accept entries of any size, but the Storage Engine was unable to handle very large entries. The worker threads in the server were trying to allocate arrays as big as the entries they were receiving to buffer their content, and then were passing them to the write buffer. I performed a few tests with all clients sending very large files, and of course, crashed the server with a nice Out Of Memory error. But the problem is not only that this design can eat up too much memory, it is also that if the entries are even of medium size, copying their data around can just take too long and end up making the network clients timeout. It is at that moment that I introduced the notion of “part” in the Write Buffer and the Storage Engine, and that I realized that because I wanted KingDB to accept entries of any size through a network, I would have to implement a Multipart API. In retrospect that seems obvious, but while I was programming, it was not.

When the database is embedded, there are no timeout constraints: in case the process embedding the database is trying to store a very large entry, it will just block on I/O and resume control when the entry has been persisted to disk. But when using the database through a network, the client cannot wait for a large entry to be persisted, as this will cause a timeout and make the entire transfer fail. The solution to this problem is to offer a Multipart API, and to cut large entries into smaller piece, which I call “parts”. The database knows what entries are currently being received and which clients are sending them. Entries are written part by part, as they come. From there it is even possible to play with the size of those parts and see what value brings in the best performance.

If there are few concurrent clients, then larger part sizes will allow for larger buffers to be used by recv(), and then sent to the Storage Engine, which means that they will incur fewer calls the write() syscall. But as the number of concurrent threads increases, the memory footprint of the program could explode. With smaller part sizes, more write() syscalls are incurred, but the memory footprint is reduced.

## 7. Write buffer and the Order class

The writes are cached in memory in the write buffer, and when there is enough of them, they are written to disk in batch. The write buffer is here to turn any workload of small random writes into a workload of large sequential writes. Large writes also have caveats and can cause write stalls as they trash the caches of the CPU and disks, but that is another story [[3, 4]](https://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/#ref).

The write buffer needs to be locked before it can be flushed. Thus if KingDB had only one write buffer, the flushing would block incoming operations. For that reason, the write buffer contains not just one, but two buffers: at any given time, one buffer has the role of receiver for incoming requests, and another buffer has the role of source for the flush operation. When the buffer used for incoming requests is ready to be flushed, its role is swapped with the buffer that was previously used to flush. The buffer with the latest requests can therefore be locked and flushed without blocking the flow of incoming requests, which are persisted to the other buffer.

The two std::vector in the Write Buffer are storing instances of the Order class. Each order contains the key of the entry it belongs to, a part of data for the value of that entry, and the offset of that part in the value array. Keys and parts are instances of ByteArray, which allows to share allocated memory buffers when they are needed all along the persisting pipeline, and seamlessly release them once they have been persisted by the Storage Engine.

## 8. Threads and synchronization

Let’s imagine that all the code were running in a single thread. At some point, a client thread would try to do a small Put() but would block due the write buffer persisting its data to disk and waiting on I/O for example, when actually those two operations could have been running in parallel. The obvious solution is to assign one dedicated thread to every independent operation that is likely to wait on I/O or that is expected to be time consuming. That way, even when one of those threads waits on I/O, the others can still make progress. There will still be moments when a thread is waiting for another one to finish, but overall, downtimes are reduced.

Because those threads are a breakdown of a larger process, they need to be synchronized, and for this I am simply using the std::mutex from the standard C++11. When I started the design of KingDB and thought about how to coordinate those threads, it was thinking of using lock-free algorithms. But because KingDB is a persisted key-value store, the bottlenecks are going to be in the file I/O, therefore even though there are clear benefits to lock-free algorithms when using hundreds of concurrent threads, using lock-free solutions did feel like over-engineering. Thus I have decided to do the first implementation simply using the standard std::mutex offered by C++11, and see how far I could get with that.

The threads used by KingDB are the following (they are also represented in Figure 8.1):

In the Write Buffer:

- **Buffer Manager**: monitors the states of the in-memory write buffers, and send one of them to the Storage Engine when its size in bytes reaches a certain threshold.

In the Storage Engine:

- **Entry Writer**: waits on EventManager::flush_buffer and processes a vector of Orders incoming from the Write Buffer
- **Index Updater**: waits on EventManager::update_index and updates the index when needed
- **Compactor**: at regular interval, it checks various metrics and determines if compaction needs to happen on the database. If so, it calls Compaction() and proceeds to compact data on disk to reclaim disk space.
- **System Statistics Poller**: at regular interval, it polls various system calls to get system metrics (free disk space, etc.)

Some of the threads need to communicate data and this is when issues of coupling can arise in the code. One of the solution I am using to avoid strong dependencies is a minimal message library which I have built. I am saying library but it’s really just 70 lines of code in a single class. It is built on top of std::condition_variable and std::mutex from the C++11 standard library, and can be found in the source code at [thread/event_manager.h](https://github.com/goossaert/kingdb/blob/master/thread/event_manager.h).

For each waypoint where two threads need a connection to pass data among them, a dedicated instance of the Event class is used. Another class, EventManager, embeds all the instances of Event that are needed for a single instance of a KingDB database:

- **flush_buffer**: Used to pass a vector of Orders from the Write Buffer to the Storage Engine. (Flush Start event in Figure 8.1)
- **update_index**: Used to pass a list of locations and hashed keys that were persisted to disk by the Storage Engine to the Index. (Index Update event in Figure 8.1)
- **clear_buffer**: Used to indicate to the Write Buffer that the last batch of Orders was saved in the Index. (Flush End event in Figure 8.1)
- **compaction_status**: Used by the user-triggered compaction to send a confirmation when it has finished. (This event was not represented in Figure 8.1)

Finally, note that no synchronization is needed to access the data persisted by KingDB in secondary storage. Indeed, the internal algorithms and log-structure storage in KingDB guarantee that no locations in any file can be at the same time used for reads and writes. Therefore, the data in HSTable files can be accessed without having to use locks.

## 9. Error management

I had covered various error management techniques in a previous article of the IKVS series [[5]](https://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/#ref). I have chosen to use a simple Status class as the error management scheme for KingDB, as it is done by LevelDB, which is itself derived from Google’s C++ style guide and their entire code base [[6]](https://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/#ref). All methods return this class and can pass to their caller an error message pointing to the exact root cause of a failure. It’s basically integer error codes on steroids.

The return values of all system calls are tested and handled appropriately. On the other hand, errors during memory allocation are not handled, and this is done on purpose. Indeed, if the program gets out of memory, there is little it can do at this stage in order to recover, so the policy is just to let it die. I have put KingDB through Valgrind for memory leaks, heap corruption, data races, etc., and all the bugs that could be found have already been fixed.

Because KingDB is implemented in C++, an obvious question is why not use exceptions? The reason why I rejected the use of exceptions is because I wanted KingDB to be optimized for readability, and exceptions work against that goal as they introduce an additional layer of complexity. In his presentation [[6]](https://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/#ref), Titus Winters arguments in favor of a Status class and against exceptions, which makes sense in the case of KingDB:

- For the readers of the source code, the Status class makes it easy to see when something can go wrong and how you’re handling it, or to see that you are explicitly not handling it
- Exceptions are hard to maintain: imagine adding or changing additional exception types for error cases you hadn’t thought about before
- Code locality matters, and exceptions make everything they can to make that not the case — technically, exceptions are free as long as they are not thrown; when they’re thrown, they incur a 10-20x slowdown with the Zero-Cost model [[7]](https://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/#ref).

## 10. Parametrization and Logging

Parametrization follows the design of LevelDB and is done through a set of classes, and parameters are set by setting attributes of these classes. Objects of these parameter classes are created and passed to the database Open() method, along with the Put() and Get() methods. Four independent classes control the parameters:

- **DatabaseOptions** is controlling all the parameters impacting the database, such as internal buffer sizes, HSTable sizes, timeout durations, etc.
- **ReadOptions** is controlling the parameters that can be tweaked when reading an entry, such as whether or not the checksum needs to be verified.
- **WriteOptions** is controlling the parameters that can be tweaked when writing an entry, such as syncing the data to secondary storage for example.
- **ServerOptions** is controlling the parameters related to the network server, such as the recv() buffer size, the listening port, etc.

The advantage of having the parameters in their own respective classes is first that the overall design is more modular, and second that parameters objects can be reused across different databases and method calls.

Logging is done through a set of methods that have the same prototype as printf(), each for a different level of alerting. That way, it is easy to control the granularity of details that one wants to display by simply changing the value of the loglevel parameter. When developing and debugging, I often use the “trace” level, which is the lowest level possible and displays all messages. When running the database in production, the “error” level, or any higher level is recommended, as it will only display messages related to errors and critical events.

## 11. Compression, checksum, and hashing

For compression, only the LZ4 algorithm is currently available: it is very fast and has proven to provide high compression ratios. As for the checksum, only CRC32 is currently available. And regarding the hashing functions used to compute the hashed keys in the index, the user can choose between the 64-bit Murmurhash3, and the 64-bit xxHash.

For all those algorithms, I have reused existing implementations with compatible software licenses, and I have imported their source code into KingDB. Thus when compiling KingDB, everything is ready, and there are no dependencies.