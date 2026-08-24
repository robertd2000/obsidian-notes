This is Part 9 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts. In this series of articles, I describe the research and process through which I am implementing a key-value database, which I have named “KingDB”. The source code is available at [http://kingdb.org](http://kingdb.org/). Please note that you do not need to read the previous parts to be able to follow. The previous parts were mostly exploratory, and starting with [Part 8](http://codecapsule.com/2015/05/25/implementing-a-key-value-store-part-8-architecture-of-kingdb/) is perfectly fine.

In this article, I explain how the storage engine of KingDB works, including details about the data format. I also cover how memory management is done through the use of a compaction process.

## 1. Single-file vs. multiple-file storage engines

When I started to work on KingDB, one of the important design decision that I had to make was to choose between a single-file database or a multiple-file database. The first difference between the two solutions is the number of inodes and system calls needed. With a single-file database, only a single inode and a single call to open() are needed. With a multiple-file database, obviously more inodes are needed, every read might require that an open() system call be done, and the kernel will need to maintain more file descriptors. Because system calls are expensive, my first design iteration was in the direction of the single-file design.

I took a pile of white paper sheets, and I started to draw random sequences of write and delete operations, sketching what the database would be, and what a compaction process would look like for that single file. This is when I became aware of a few interesting cases. For example, imagine that a very large entry is written right in the middle of that single-file database, and that later on, a delete operation was emitted for that same entry: the disk space used by that entry will have to be reclaimed somehow. One option for that is to shift up the rest of the file: that operation is going to be very costly if the database file is very big, thus it is not feasible. Another option is to mark the space previously used by the deleted entry as “free” space, so that incoming writes could use it. With this option, the storage engine would need to create blocks of memory, allocate those blocks for entries, and keep track of which of these blocks are used and which are free. So essentially, the storage engine would have to implement a memory allocator or a file system. Ouch! Having to implement a memory management system was going to add a lot of complexity to the code, and this is exactly what I wanted to avoid in my design. At this point, I came to another realization: why implement a file system when the operating system is giving you one? And so I started thinking about how I could solve this storage problem using a multiple-file approach.

With multiple files, the compaction process is going to be greatly simplified: just combine uncompacted files, the ones that have deleted or overlapping entries, and store them into new compacted files. Then simply delete the uncompacted files, and the file system will reclaim the free disk space. And when some extra space is needed, the program just has to create a new file, and the file system takes care of everything: allocation, fragmentation, free space management, etc. Keeping in mind that KingDB needs to handle small entries just as well as large entries, I still had to find what the data format for this multiple-file database would be like. After a few iterations, I reached a solution that allows for all entry sizes to be handled properly, while still keeping the overall design simple. One important detail: most file systems do poorly with many small files, so the multiple-file design would need to be made such that the size of those files could be parametrized so they can be made large enough to be made efficient for a file system.

## 2. Data Format: Hashed String Tables (HSTables)

The data format used by KingDB is the Hashed String Table, or HSTable. Don’t google that term, you won’t find anything about it: I developed this data format and made up its name as I designed KingDB. It’s just a data format that stores key-value type binary data.

Each HSTable file starts with a header at position 0, and the first 8192 bytes are reserved for the header. The entries start at position 8192, and are stored contiguously. Each entry starts with an Entry Header that contains some metadata, and is followed by the sequence of bytes for the key and the value of the entry. At the end of each HSTable, an Offset Array stores a compact representation of the entries stored in the HSTable: for each entry in the HSTable, the Offset Array has one row which is the hashed key of that entry, and the offset where the entry can be found in the file. The Offset Array can be used to quickly build a hash table in memory, mapping hashed keys to locations in HSTables.

Here is what a typical HSTable looks like:

[HSTableHeader]
[zero padding until byte 8192]
[EntryHeader for entry 0] 
   [byte sequence for the key of entry 0]
   [byte sequence for the value of entry 0]
   [zero padding for the value of entry 0 if compression enabled]
   ...
[EntryHeader for entry N-1] 
[byte sequence for the key of entry N-1]
[byte sequence for the value of entry N-1]
[zero padding for the value of entry N-1 if compression enabled]
[Row 0 of the OffsetArray: hashed key of entry 0 and offset to entry 0]
...
[Row N-1 of the OffsetArray: hashed key of entry N-1 and offset to entry N-1]
[HSTableFooter] 

And below is the content of the various parts of an HSTable presented above:

**/* HSTableHeader */**

uint32_t checksum;                     // 32-bit checksum for the HSTable header
uint32_t version_major;                // Data format version (major)
uint32_t version_minor;                // Data format version (minor)
uint32_t version_revision;             // Data format version (revision)
uint32_t version_build;                // Data format version (build)
uint32_t version_data_format_major;    // Data format version (major)
uint32_t version_data_format_minor;    // Data format version (minor)
uint32_t filetype;                     // whether the file is regular or large,
                                       // and whether or not it was already compacted
uint64_t timestamp;                    // Timestamp of the HSTable, i.e. the order in which this
                                       // HSTable must be considered to guarantee the order
                                       // of operations in the database

**/* EntryHeader */**

uint32_t checksum_header;        // Checksum for the EntryHeader
uint32_t checksum_content;       // Checksum for the sequence of bytes across entry key and value data
uint32_t flags;                  // Flags, such as: kTypeRemove, kHasPadding, and kEntryFull
uint64_t size_key;               // Size of the key (in bytes)
uint64_t size_value;             // Size of the value when uncompressed (in bytes)
uint64_t size_value_compressed;  // Size of the value when compressed,
                                 // or 0 if compression is disabled (in bytes)
uint64_t size_padding;           // Size of the padding at the end of the entry, if any (in bytes)
uint64_t hash;                   // Hashed key

**/* OffsetArray Row */**

uint64_t hashed_key;      // hashed key of the entry
uint32_t offset_entry;    // offset where the entry can be found in the HSTable 

**/* HSTableFooter */**
uint32_t filetype;        // Same as filetype in HSTableHeader
uint32_t flags;           // Flags, such as: kHasPaddingInValues, and kHasPaddingInValues
uint64_t offset_offarray; // Offset of the OffsetArray in the HSTable
uint64_t num_entries;     // Number of entries
uint64_t magic_number;    // Magic number
uint32_t checksum;        // Checksum for the rows in the Offset Array and the HSTableFooter

For the headers and footers above, all the data is serialized in little-endian for cross-platform compatibility. Some of the fields, such as the sizes in the EntryHeader, are stored using Variable-Length Integers, “varints”, which allow for a compact representation of integers and save a significant amount of disk space [[1, 2]](https://codecapsule.com/2015/08/03/implementing-a-key-value-store-part-9-data-format-and-memory-management-in-kingdb/#ref).

Also, note that the checksums are computed for each entry independently. The upside is that since each entry has its own checksum, and a wrong checksum means that only that entry will be deleted. The downside is that when entries are small, this ends up using more disk space.

## Join my email list

## 3. KingDB’s Storage Engine

I designed the storage engine with the network in mind. Chunks of data received from the network by a call to recv() are saved into a buffer first, and then persisted to disk. If the size of this buffer is too large, then writing the data to secondary storage can take more time than the acceptable network inactivity delay, which will make the connection timeout. Therefore, the size of the recv() buffer must kept small, and a good practical size for that is 8KB.

KingDB stores data in HSTable files, as presented in Section 2. Those HSTable files have a maximum size of 256MB by default — this is a parameter that can be changed when creating a database. Assuming that the recv() buffer of the server is 8KB, there are three types of entries:

- **Small entries:** smaller than the recv() buffer, thus size <= 8KB
- **Medium entries:** larger than the recv() buffer, but smaller than the size of the HSTable files: 8KB < size <= 256MB
- **Large entries:** larger than the size of a HSTable file: size > 256MB

When small entries are incoming, they are copied into a buffer before being persisted to disk, into the next new created HSTable. This means that workloads of small random writes are turned into a workload of large sequential writes.

When medium entries arrive, their first part is copied to the buffer, and then some space is reserved on disk so that the subsequent parts can be written contiguously. Each subsequent part requires a call to pwrite(). Another option would have been to allow for parts to be stored in different files, but then with a slow client, the database could end up with a situation where the N parts of an entry are stored at random locations in N different files, which makes compaction obviously more complex. Keeping all parts for a given entry contiguous simplifies the compaction process.

Finally, when a large entry is incoming, it is given its own dedicated file, and every part is persisted to disk with its own call to pwrite().

Of course, the use of multiple pwrite() syscalls is not optimal, but it guarantees that workloads of concurrent small, medium, and large entries, can be handled. In addition, benchmarks over the write syscall shows easily that past a certain buffer size, the cost of the syscall is amortized, thus by picking the size of the recv() buffer to be large enough, 8KB, the cost of the additional calls to pwrite() are amortized [[3]](https://codecapsule.com/2015/08/03/implementing-a-key-value-store-part-9-data-format-and-memory-management-in-kingdb/#ref).

Figure 9.1 below illustrates what the storage engine looks like when it is receiving buffers of entry parts to write to disk. This is a purely random sequence of entries that I have made up so I could show how the storage engine works. Entries are represented with a unique lowercase letters and a distinct color. In this simplified representation, the buffer is of size 6 and the HSTable have a maximum size of 8. All entries have integer number sizes, and all the HSTables are filled exactly up to the maximum HSTable size. This maximum HSTable size only applies to small and medium entries, and can of course be extended in the case of large entries as explained above. In the real implementation of KingDB, HSTables can have sizes a bit lower or a bit larger than the maximum HSTable size, depending on the sizes of the incoming entries they have to hold.

![storage-engine-kingdb-0.9.0](https://i0.wp.com/codecapsule.com/wp-content/uploads/2015/08/storage-engine-kingdb-0.9.0.png?resize=780%2C1681)

Figure 9.1: Storage Engine of KingDB v0.9.0

## 4. Database start-up, recovery and index building

When a database is opened, its parameters are read from the option file, so that the appropriate hashing, compression, and checksum functions can be created. Then the HSTables are ordered by timestamp first and file id second, and handled one after the other.

For each HSTable, the HSTableFooter is read:

- If anything is wrong with the footer, i.e. invalid magic number or invalid checksum, the HSTable enters recovery mode. In recovery mode, all the entries are read from the header one by one, and their checksum are computed to verify their integrity: invalid entries are discarded.
- If the footer is valid, the position of the Offset Array is read from offset_offarray, and all the items of that Offset Array are loaded into the in-memory hash table of KingDB. Building the index requires only a single bulk sequential read in each HSTable. With this data format and the Offset Array in each HSTable, building the entire index is fast, and thus the start-up time is very small: an average-size database can be loaded and used after just a few seconds.

## 5. Compaction

KingDB uses log-structured storage to persist entries to disk. With that solution, if multiple versions of an entry — i.e. entries with same key — are saved in the database, all these versions will be stored on disk, but only the last one has to be kept. Therefore a compaction process must be applied to the entries so that the space occupied by outdated entries can be reclaimed. It is frequently reported that storage systems undergo slowdowns when the compaction process kicks in. This is nonetheless the design that I have chosen to use for KingDB, because of its simplicity.

Compaction is performed at regular time interval if certain conditions are met. The compaction process looks at the uncompacted files in the same order as they were written, and based on the currently available free disk space, it determines which subset of these files can be compacted. For every uncompacted file, the compaction process checks if for any of the entries in these files, there exists an older version of those entries in an already compacted file, i.e. overwrite of entries with identical keys. The compaction also looks at the delete orders, and checks if it can find a file that contains the data for the deleted key. If such combinations of files are found, then these files will be merged into a single sequence of orders, and written to a sequence of new compacted files. In the mean time, the stale versions of the entries and the deleted entries, the uncompacted files, are discarded. When the new compacted version of the files have been written to disk, the uncompacted files are simply removed. The compaction process never overwrites any data, it always write data to a new file, at a new location, guaranteeing that if an error or a crash occurs, it will not cause any data loss. In addition, compaction is designed in such a way that if cursors or snapshots are requested by a client as a compaction is on-going, the files that the client may use are “locked” until the cursors and snapshots are released, at which point they will be removed.

During compaction, an important process also happens: key grouping. Whenever the compaction process encounters entries that have the same hashed keys but different keys — i.e they are not the successive versions of the same entry, they are effectively different entries — it writes them sequentially in the new sequence of files. This can incur a fair bit of rewrites, but it guarantees that after all files have gone through the compaction process, all entries that have the same hashed key will be found sequentially in files. Therefore, whenever hashed key collisions happen in the index and whenever the same hash points to different entries, only a single seek on disk will be necessary to access the whole set of entries having that hash: all entries can be found with a single random read. For SSDs there are no seeks of course, but this will still guarantee that data access can be made using the internal read-ahead buffers of the drives.

Figure 9.2 below shows what a compaction process looks like on an arbitrary set of HSTables.

![compaction-kingdb-0.9.0](https://i0.wp.com/codecapsule.com/wp-content/uploads/2015/08/compaction-kingdb-0.9.0.png?resize=780%2C2187)