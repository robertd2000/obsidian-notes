This is Part 7 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts. In this series of articles, I describe the research and process through which I am implementing a key-value store database, which I have named “KingDB”.

In the previous articles, I have spent a fair amount of time reviewing existing key-value stores, interfaces, architectures, and I focused greatly on hash tables. In this article, I will talk about hardware considerations for storing data structures on solid-state drives (SSDs), and I will share a few notes regarding file I/O optimizations in Unix-based operating systems.

This article will cover:

1. Fast data structures on SSDs  
2. File I/O optimizations  
3. Done is better than perfect  
4. References

## Join my email list

## 1. Fast data structures on SSDs

As I was researching resources on efficient ways to implement KingDB, I fell down a gigantic rabbit hole: solid-state drives. I have compiled the results of that research into another series of articles, _“Coding for SSDs”_ [[1]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref), and I have summarized the most important points in the last article of that series: _“What every programmer should know about solid-state drives”_ [[3]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref). I encourage you to read the whole series if you have time, and if not, then at least have a sneak peak at the summary. All that research and this series of articles on SSDs were necessary, as they gave me the confidence that my design decisions for KingDB were taking me in the right direction.

### In-place updates are useless

SSDs differ dramatically from regular HDDs. The most important thing about SSDs is that, unlike HDDs, nothing is ever overwritten. Updates to an existing page are applied in an internal register, a RAM module in the SSD, and the updated version of the page is stored at a different location on the SSD. The original page is considered “stale” until the garbage collection process erases it and puts it back in the pool of free pages. As a direct consequence, any algorithm trying to be clever by applying updates in-place will bring no performance gain, and will needlessly increase code complexity.

### Metadata must be buffered or avoided

The smallest size of data that can be written to an SSD is a NAND-flash page — 4 KB, 8 KB, or 16 KB depending on the SSD model — which can have serious consequences for any metadata stored on the drive. Imagine that a hash table needs to persist on the drive the number of items it is currently holding, as a 64-bit integer. Even though that number is stored over 8 bytes, every time it is written to the drive an entire NAND-flash page will be written. Thus an 8-byte write turns into a 16 KB write, every single time, leading to dramatic write amplification. As a result, when stored on SSDs, data structures need to cache their metadata and write them to the drive as infrequently as possible. Ideally, the file format will be designed in a way that this metadata never needs to be updated in-place, or even better, that this metadata never needs to be stored at all [[2]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref).

### Data is never really sequential

In SSDs, addresses that are sequential in the logical space do not correspond to addresses that are sequential in the physical space. This means that if you open a file and write sequentially to it, the data is not stored sequentially on the drive. Of course, if data is written sequentially within the alignment of a NAND-flash page, then it will be sequential on the SSD, but two pages that are sequential in the logical space have very little chances to be stored sequentially in the drive. Thus for chunks larger than the size of the NAND-flash page, rearranging data sequentially hoping to optimize for future reads will brings no improvements. Not only that, but the amount of data moved around eats up CPU time, and causes the NAND-flash memory modules in the SSD to wear off prematurely. Below the size of the NAND-flash page, storing data sequentially in the logical space still helps.

### Random writes are fast if they are large enough

SSDs have several levels of internal parallelism. A “clustered block” is a set of NAND-flash memory modules that can be accessed concurrently, and has a size of 16 MB or 32 MB for most SSDs. If writes are performed in the size of at least one clustered block, they will use all the levels of internal parallelism, and reach the same performance as sequential writes [[2]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref).

### Random reads are fast

The data written together in a buffer the size of a clustered block will use all the internal parallelism of an SSD. If the same data is later retrieved, the throughput will be very high because the SSD will be able to fire up multiple memory modules at the same time, using of all its internal parallelism. For most workloads, it is impossible to predict what data will be read, and therefore, the database cannot take advantage of this aspect of the SSD architecture. However, read performance in SSDs is very good on average, whether reads are sequential or random. Therefore the focus for data structures persisted on SSD should really be on optimizing for the writes rather than for the reads.

### Log-structured storage

The goal that I set myself for KingDB is to minimize the number of writes performed to the SSD incurred by incoming updates. The best option to cope with the specificities of SSDs that I have just mentioned above is to use the “log-structured” paradigm. Originally developed for file systems [[4]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref), more data structures have started to use that approach. One of the most famous is now the Log-Structured Merge-Tree (LSM tree), which is the data structure at the heart of the LevelDB key-value store [[5]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref).

The idea behind log-structured storage is simple: treat the incoming updates as a log stream, and write them on the drive sequentially, exactly as they come. Let’s take an example to make things clearer. Assume that a database has an entry with key “foo”, and that a new update, a Put(), comes for the same key: the old value for that key needs to be replaced with the new value. What log-structured storage does is that it keeps the old value where it is, and store the new value in the free space available just after the last incoming update. Then the index is updated to make the key “foo” point to the new location. The space used by the old value will be later recovered during a compaction process. Again, note that the old value is not changed in-place, and this is good because SSDs cannot do in-place updates anyway.

Another interesting side effect of log-structured storage is that, because it allows for writes to be as large as one wants them to be, they can be in the order of magnitude of the clustered block (32 MB). Incoming random updates can be batched into a single large write operation, by buffering them into the RAM for example, effectively turning a workload of small random writes into a workload of large sequential writes.

## 2. File I/O optimizations

### Every syscall has a cost

The days when every system call was causing a full context switch are long gone. Nowadays, going from user land into kernel space only requires a “mode switch”, which is much faster as it does not require the CPU caches and TLB to be invalidated. Those mode switches still have a cost, roughly 50 to 100 ns depending on the CPU [[6]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref).

In Chapter 13 of his excellent book “The Linux Programming Interface”, Michael Kerrisk benchmarked the time it takes to write 100 MB of data with different buffer sizes. In one of his tests, it took 72 seconds to write 100 MB with a buffer size of 1 byte, and only 0.11 second to write the same 100 MB with a buffer size of 4 KB. No actual file I/O was performed as the changes were not synced to secondary storage, thus the test was effectively measuring the time spent in the system calls. This is why minimizing the number of system calls and determining the correct sizes for the buffers is crucial to performance.

### Memory maps

Memory maps are the best thing known to mankind after hash tables. A memory map is basically a mapping of an area of a file into an address in the memory. That address can then be accessed, for both reads and writes, using a pointer like any regular buffer of allocated memory, and the kernel takes care of actually reading and writing the data to the file on the drive. There are countless success stories of improvements achieved by replacing calls to read() and write() with memory maps, an interesting one being covered in the SQLite documentation [[7]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref). Optimizing file I/O is hard, and most of the time, is better left to the kernel. As a consequence, I will try to use memory maps as much as possible for KingDB. I will indulge myself the option of using calls to read() and write() if I judge that it can speed up the development time. Vectored I/O, also known as scatter-gather I/O, is another tool worth mentioning as it can offer substantial performance improvements, and deserves a try whenever applicable [[8]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref).

### Zero-copy

Most system calls copy data from a buffer in user land to a another buffer in kernel space, or conversely, causing a lot of unnecessary data duplication. Zero-copy is there precisely to minimize the amount of data that needs to be copied around, with the goal of copying absolutely zero byte between user space and kernel space. The sendfile() syscall, which sends a file to a socket, is a typical example of zero-copy transfer. A buffer is still required for sendfile(), as pages from the file need to be loaded into RAM before they are sent to the socket, but this buffer remains in kernel space, and there is no data copy between user and kernel space [[9, 10, 11]](https://codecapsule.com/2014/10/18/implementing-a-key-value-store-part-7-optimizing-data-structures-for-ssds/#ref). Memory maps are also performing zero-copy operations, thus it is good to stick to memory maps and not rush into using specific system calls such as sendfile() and splice().

Data copying is a very serious source of wasted time in a database system, as the total time spent needlessly duplicating data around can sum up to large latencies. It can happen anywhere, not only when using syscalls, and must be monitored closely.

## 3. Done is better than perfect

In the two sections above, I have covered the most important points regarding the storing of data structures on SSDs and file I/O optimizations. I want to consider those sections only as a general direction, from which I can steer away at any moment if I gather data to prove that other options are preferable. Also, it is fine to cut corners, because the goal for me is to get a working key-value store as soon as possible: done is better than perfect.

I have started implementing KingDB, the key-value store related to this series of article. The source code, still under development, is available at [http://kingdb.org](http://kingdb.org/). In the next article, I will review the architecture of KingDB and its components.

To receive a notification email every time a new article is posted on Code Capsule, you can subscribe to the newsletter by filling up the form at the top right corner of the blog.  
As usual, comments are open at the bottom of this post, and I am always happy to welcome questions, corrections and contributions!

## Join my email list

## 4. References

[1] [Coding for SSDs – Part 1: Introduction and Table of Contents](http://codecapsule.com/2014/02/12/coding-for-ssds-part-1-introduction-and-table-of-contents/)  
[2] [Coding for SSDs – Part 5: Access Patterns and System Optimizations](http://codecapsule.com/2014/02/12/coding-for-ssds-part-5-access-patterns-and-system-optimizations/)  
[3] [Coding for SSDs – Part 6: A Summary – What every programmer should know about solid-state drives](http://codecapsule.com/2014/02/12/coding-for-ssds-part-6-a-summary-what-every-programmer-should-know-about-solid-state-drives/)  
[4] [Log-structured File System](http://en.wikipedia.org/wiki/Log-structured_file_system)  
[5] [SSTable and Log Structured Storage: LevelDB](https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/)  
[6] [How long does it take to make a context switch](http://blog.tsunanet.net/2010/11/how-long-does-it-take-to-make-context.html)  
[7] [SQLite Documentation: Memory-Mapped I/O](https://www.sqlite.org/mmap.html)  
[8] [Fast Scatter-Gather I/O](http://www.gnu.org/software/libc/manual/html_node/Scatter_002dGather.html)  
[9] [Zero-Copy Sockets](https://www.cs.duke.edu/ari/trapeze/freenix/node6.html)  
[10] [Efficient data transfer through zero copy](http://www.ibm.com/developerworks/library/j-zerocopy/)  
[11] [Zero Copy I: User-Mode Perspective](http://www.linuxjournal.com/article/6345)