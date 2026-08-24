This is Part 5 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts.

In this article, I will study the actual implementations of hash tables in C++ to understand where are the bottlenecks. Hash functions are CPU-intensive and should be optimized for that. However, most of the inner mechanisms of hash tables are just about efficient memory and I/O access, which will be the main focus of this article. I will study three different hash table implementations in C++, both in-memory and on-disk, and take a look at how the data are organized and accessed. This article will cover:

1. Hash tables  
    1.1 Quick introduction to hash tables  
    1.2 Hash functions  
2. Implementations  
    2.1 unordered_map from TR1  
    2.2 dense_hash_map from SparseHash  
    2.3 HashDB from Kyoto Cabinet  
3. Conclusion  
4. References

![](https://i0.wp.com/codecapsule.com/wp-content/uploads/2013/05/kvstore-part5-intro.jpg?resize=770%2C315)

## 1. Hash tables

### 1.1 Quick introduction to hash tables

> Hashtables are arguably the single most important data structure known to mankind.  
> — Steve Yegge

A hash table allows to efficiently access _associative data_. Each _entry_ is a pair of a _key_ and a _value_, and can be quickly retrieved or assigned just by knowing its key. For that, the key is hashed using a _hash function_, to transform that key from its original representation into an integer. This integer is then used as an index to identify the _bucket_ in the _bucket array_ from which the entry’s value can be accessed. Many keys can hash to the same values, meaning that these keys will be in _collision_ in the bucket array. To resolve collisions, various techniques can be used, such as _separate chaining_ with linked-lists or self-balanced trees, or _open addressing_ with linear or quadratic probing.

From now on, I will assume that you know what hash tables are. If you think you need to brush up your knowledge a bit, good references are either the “Hash table” article on Wikipedia [[1]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_1) (and the external links section at the bottom of the page), or the Hash table chapter in the book “_Introduction to Algorithms_” by Cormen et. al [[2]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_2).

### 1.2 Hash functions

The choice of the hash function is extremely important. The basic requirement for a good hash function is that the output hashed values should be distributed uniformly. That way, the chances of collisions are minimized, along with the average number of colliding entries in a bucket.

There are many possible hash functions, and unless you know exactly what the data are going to be, the safest option is to go for a hash function that distributes random data uniformly on average, and if possible that fits the _avalanche effect_ [[3]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_3). A few people have already worked on hash function comparison [[4]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_4) [[5]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_5) [[6]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_6) [[7]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_7), and from their conclusions, it is clear that MurmurHash3 [[8]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_8) and CityHash [[9]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_9) are the best hash functions to use for hash tables at the time this article is being written.

## 2. Implementations

Like for the comparisons of hash functions, there are a few blog articles that already compare the performance of in-memory C++ hash table libraries. The most notables I have encountered are “_Hash Table Benchmarks_” by Nick Welch [[10]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_10) and “_Hash Table Performance Tests_” by Jeff Preshing [[11]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_11), but other articles also deserve a glance [[12]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_12) [[13]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_13) [[14]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_14). From these comparisons, I have derived that unordered_map from TR1 in GCC along with dense_hash_map from the SparseHash library — formerly called Google SparseHash — are two interesting pieces to study, and I will cover them below. In addition, I will also describe the data structures inside HashDB from Kyoto Cabinet. Obviously, unordered_map and dense_hash_map won’t be as relevant as HashDB for my key-value store project, since they are in-memory hash tables. Nevertheless, having a glance at how their inner data structures are organized and what are the memory patterns can only be interesting.

For the descriptions of the three hash table libraries below, I will take as a common example a set of city names as keys, and their GPS coordinates as values. The source code for unordered_map can be found in GCC’s code, as part of libstdc++-v3. I’ll be looking at libstdc++-v3 release 6.0.18 from GCC v4.8.0 [[15]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_15), dense_hash_map from SparseHash v2.0.2 [[16]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_16), and HashDB from Kyoto Cabinet v1.2.76 [[17]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_17).

Interesting implementation discussions can also be found in “_A Proposal to Add Hash Tables to the Standard Library (revision 4)_” by Matthew Austern [[18]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_18) and in the “_Implementation notes_” page of SparseHash [[19]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_19).

### 2.1 unordered_map from TR1

TR1’s unordered_map provides a hash table that handles collisions with linked lists (separate chaining). The bucket array is allocated on the heap, and scales up or down automatically based on the load factor of the hash table. A node struct named `_Hash_node` is used to create the linked lists for the buckets:

**/* from gcc-4.8.0/libstdc++-v3/include/tr1/hashtable_policy.h */** 
template
  struct _Hash_node<_Value, false>
  {
    _Value       _M_v;
    _Hash_node*  _M_next;
  };

If the keys and values are of integral types, they can be stored directly inside this struct in `_M_v`. Otherwise pointers will be stored and some extra memory will be necessary. The bucket array is allocated at once on the heap, but it’s not the case of the Nodes, which are allocated with individual calls to the C++ memory allocator:

**/* from gcc-4.8.0/libstdc++-v3/include/tr1/hashtable.h */** 
Node* _M_allocate_node(const value_type& __v)
    {
      _Node* __n = _M_node_allocator.allocate(1);
      __try
	{
	  _M_get_Value_allocator().construct(&__n->_M_v, __v);
	  __n->_M_next = 0;
	  return __n;
	}
      __catch(...)
	{
	  _M_node_allocator.deallocate(__n, 1);
	  __throw_exception_again;
	}
    }

Because nodes are allocated individually, a lot of memory may be wasted on every node allocation. This depends of course on the memory allocator of the compiler and operating system being used. And I am not even talking about all the system calls being performed for each allocation. The original implementation of the SGI hash table was doing some resource pre-allocation for the nodes, but this solution has not been kept for the unordered_map implementation of TR1.

Figure 5.1 below offers a representation of the memory and access patterns for unordered_map from TR1. Let’s see what happens if we look for the GPS coordinates associated with the key “Johannesburg”. This key is hashed and mapped to the bucket #0. From there we jump to the first node of the linked list for that bucket (orange arrow on the left of bucket #0), and we can access the memory area in the heap that holds the data for the key “Johannesburg” (black arrow on the right of the node). If the key were to be invalid at this first node, we would have had to navigate throw other nodes.

As for CPU performance, one cannot expect to have all the data in the same cache line in the processor. Indeed, given the size of the bucket array, the initial bucket and the initial node will not be in the same cache line, and the external data associated with a node is also unlikely to be found on the same cache line. Subsequent nodes and associated data will also not be in the same cache line and will have to be retrieved from RAM. If you are not familiar with CPU optimizations and cache lines, the “_CPU Cache_” article on Wikipedia is a good introduction [[20]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_20).

![kvstore_unordered_map_web](https://i0.wp.com/codecapsule.com/wp-content/uploads/2013/04/kvstore_unordered_map_web.jpg?resize=770%2C948)

Figure 5.1

### 2.2 dense_hash_map from SparseHash

The SparseHash library offers two hash table implementations, _sparse_hash_map_ and _dense_hash_map_. sparse_hash_map offers amazing memory footprint at the cost of being slow, and uses a specific data structure to achieve such results, a _sparsetable_. More information about sparsetables and sparse_hash_map can be found in the “_Implementation notes_” page of SparseHash [[19]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_19). Here I will only cover dense_hash_map.

dense_hash_map handles collisions with quadratic internal probing. Like for unordered_map, the bucket array is allocated on the heap at once, and scales up or down automatically based on the load factor of the hash table. Elements of the bucket array are instances of `std::pair` where `Key` are `T` are the template parameters for the keys and values, respectively. On a 64-bit architecture and for storing strings, an instance of pair will be 16 bytes.

Figure 5.2 below is a representation of the memory and access patterns for dense_hash_map. If we look for the GPS coordinates of “Johannesburg”, we would fall in bucket #0 at first, which has data for “Paris” (black arrow at the right of bucket #0). So we would have to probe and jump at bucket (i + 1) = (0 + 1) = 1 (orange arrow at the left of bucket #0), and then we would find the data for “Johannesburg” from bucket #1 (black arrow at the right of bucket #1). This seems similar to what was going on with unordered_map, but it is actually very different. Sure, the keys and values will have to be stored in memory allocated on the heap just like for unordered_map, which means that the key and value lookups will invalidate the cache line. But navigating among the entries in collision for a bucket is going to be rather fast. Indeed, given that each pair is 16 bytes and that the cache line is 64 bytes on most processors, the probing steps are very likely to be on the same cache line, which is going to speed things up dramatically, as opposed to the linked list in unordered_map which required jumping in the RAM to get the following nodes.

This cache line optimization offered as by the quadratic internal probing is what makes dense_hash_map the winner of all the performance tests for in-memory hash tables (as least those I have read so far). You should take a moment to review the “Hash Table Benchmarks” article by Nick Welch [[10]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_10).

![kvstore_hash_dense_hash_map_web](https://i0.wp.com/codecapsule.com/wp-content/uploads/2013/04/kvstore_hash_dense_hash_map_web.jpg?resize=770%2C910)

Figure 5.2

## Join my email list

### 2.3 HashDB from Kyoto Cabinet

Kyoto Cabinet implements many data structures, among which a hash table. This hash table, HashDB, was designed to be persistent on-disk, even though there is an option to use it as an in-memory replacement for `std::map`. The hash table metadata along with the user’s data are all stored sequentially in a unique file on disk using the file system.

Kyoto Cabinet handles collisions with separate chaining through a binary search tree for each bucket. The bucket array has a fixed length and is never resized, regardless of the state of the load factor. This has been a major drawback of the hash table implementation of Kyoto Cabinet. Indeed, if the size of the bucket array defined at the creation of the database is below its actual needs, then performance suffers badly when entries start colliding.

It is very difficult to allow the bucket array to be resized for an on-disk hash table implementation. First, that would require the bucket array and the entries to be stored into two separate files, so that they could grow independently. Second, since resizing the bucket array requires to re-hash the keys to their new locations in the new bucket array, that would require reading from disk all the keys for all the entries, which would be very costly or almost impossible in the case of very large databases. One way to avoid this re-hashing process would be to store the hashed keys, but that would mean 4 or 8 more bytes of structural data for each entry (depending on whether the hash is 32- or 64-bit long). Because of all these complications, having a fixed-length bucket array is simpler, and it is the solution that was adopted for HashDB in Kyoto Cabinet.

Figure 5.3 shows the structure of a HashDB stored in a file. I have derived this internal structure from the code in the `calc_meta()` method, and from the comments of the attributes of the HashDB class at the end of kchashdb.h. The file is organized in sections as follows:

- The headers with all the metadata for the database
- The FreeBlock pool that holds the free space in the data area
- The bucket array
- The records (data area)

A record holds an entry (key/value pair), along with a node of the binary search tree for the separate chaining. Here is the Record struct:

**/* from kyotocabinet-1.2.76/kchashdb.h */** 
  /**
   * Record data.
   */
  struct Record {
    int64_t off;                         ///< offset
    size_t rsiz;                         ///< whole size
    size_t psiz;                         ///< size of the padding
    size_t ksiz;                         ///< size of the key
    size_t vsiz;                         ///< size of the value
    int64_t left;                        ///< address of the left child record
    int64_t right;                       ///< address of the right child record
    const char* kbuf;                    ///< pointer to the key
    const char* vbuf;                    ///< pointer to the value
    int64_t boff;                        ///< offset of the body
    char* bbuf;                          ///< buffer of the body
  };

The on-disk organization of a record can be observed on Figure 5.4. I derived this organization from the code in the `write_record()` method in kchashdb.h. Note that this is different from the Record struct: the goal of the on-disk representation is to minimize space on disk, while the struct aims at making the record easy to use programmatically. All the fields in Figure 5.4 have a fixed length, except for `key`, `value`, and `padding`, which of course depend on the size of the data being held by the entry. The `left` and `right` fields are part of the node of the binary search tree, and hold the offset to other records in the file.

![kvstore_hash_kyoto_cabinet_web](https://i0.wp.com/codecapsule.com/wp-content/uploads/2013/04/kvstore_hash_kyoto_cabinet_web.jpg?resize=770%2C910)

Figure 5.3

![kvstore_hash_kyoto_cabinet_record_web](https://i0.wp.com/codecapsule.com/wp-content/uploads/2013/04/kvstore_hash_kyoto_cabinet_record_web.jpg?resize=329%2C362)

Figure 5.4

If we wanted to access the value for the key “Paris”, we would start by getting the offset of the initial record for the associated bucket, which happens to be bucket #0. We would then jump to the head node of the binary search tree for that bucket (orange arrow on the left of bucket #0), which holds the data for the key “Johannesburg”. The data for the key “Paris” can then be accessed through the right child of the current node (black arrow at the right of the record for “Johannesburg”). Binary search trees need a “comparable” type in order to classify nodes. The comparable type used here is simply the hashed keys shrunk into a smaller representation using the `fold_hash()` method:

**/* from kyotocabinet-1.2.76/kchashdb.h */** 
uint32_t fold_hash(uint64_t hash) {
  _assert_(true);
  return (((hash & 0xffff000000000000ULL) >> 48) | ((hash & 0x0000ffff00000000ULL) >> 16)) ^
      (((hash & 0x000000000000ffffULL) << 16) | ((hash & 0x00000000ffff0000ULL) >> 16));
}

Storing the entries and nodes together into a single record might seem like a design mistake at first, but it is actually very clever. In order to store the data for an entry, one will always need to manage three different data: bucket, collision, and entry. Given that buckets in the bucket array must be stored sequentially per definition, they will be stored as such and there is nothing to improve there. Then assuming we are not storing integral types but strings or variable-length byte arrays that cannot be stored in the buckets themselves, another memory access will have to be made outside of the area of the bucket array. Therefore when adding a new entry, one would need to store data for the collision data structure and for the entry’s key and value.

If the collision and entry data were stored separately, that would require accessing the disk twice, in addition to the access already required for the bucket. In the case of setting a value, that would make a total of three writes on disk, at potentially very distant locations. This means a pattern of random writes on disk, which is as far as I/O is concerned the worst possible thing ever. Now since in Kyoto Cabinet’s HashDB the node data and entry data are stored together, they can be committed to disk with just one write instead of two. Sure, the bucket still has to be accessed, but if the bucket array is small enough, then chances are that it will be cached from disk into RAM by the operating system anyway, which is one of the major assumption of Kyoto Cabinet, as stated in the Section “Effective Implementation of Hash Database” of the specs [[17]](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/#ref_).

There is however one concern to be raised with having the binary search tree nodes stored with the entries on disk, which is that it slows down the reads, at least when collisions start kicking in. Indeed, since the nodes are stored with the entries, resolving a collision in a bucket means finding the record holding the valid entry in the binary search tree, which requires possibly many random reads on the disk. This gives a better understanding as to why Kyoto Cabinet shows such drops in performance when the number of entries exceeds the number of buckets.

Finally, because everything is stored in a file, memory management is being handled by Kyoto Cabinet itself, and is not left to the operating system like it is the case for unordered_map and dense_hash_map. The FreeBlock struct holds information regarding free space in the file, which is basically the offset and size, as it can be seen here:

**/* from kyotocabinet-1.2.76/kchashdb.h */** 
  /**
   * Free block data.
   */
  struct FreeBlock {
    int64_t off;                         ///< offset
    size_t rsiz;                         ///< record size
    /** comparing operator */
    bool operator <(const FreeBlock& obj) const {
      _assert_(true);
      if (rsiz < obj.rsiz) return true;
      if (rsiz == obj.rsiz && off > obj.off) return true;
      return false;
    }
  };

All the FreeBlock instances are loaded in a std::set, which allows free memory blocks to be retrieved using the upper_bound() method of std::set as seen in the `fetch_free_block()` method, making the memory allocation strategy a “best fit”. When the free space appears to be too fragmented or that no space is left in the FreeBlock pool, the file is defragmented. This defragmentation process moves records around to reduce the overall size of the database file.

## 3. Conclusion

In this article, I have presented the data organization and memory access patterns for three different hash table libraries. The unordered_map from TR1 and dense_hash_map from SparseHash are in memory, and HashDB from Kyoto Cabinet is on disk. All three make use of different solutions for handling collisions, with different effects on performance. Separating the bucket data, collision data and entry data will impact performance, which is what happens with unordered_map. Speed can be improved greatly by storing the collision data with either the buckets, as it is the case with dense_hash_map and its quadratic internal probing, or with the entries as it is the case with HashDB. Both solutions improve the speed for the writes, but storing the collision data with the buckets will also makes the reads faster.

If there is one thing that I have learned from studying those hash table libraries, it is that when designing the data organization of a hash table, the preferred solution should be to store the collision data with the buckets and not with the entries. This is because even if the hash table is on disk, the bucket array and collision data will be small enough so that they can be stored in the RAM, where random reads are a lot cheaper than on disk.

## Join my email list

## Translations

This article was translated to [Simplified Chinese](http://blog.xiongduo.cn/IT-Trans/ikvs-5-hash-table-implementations.html) by Xiong Duo.

## 4. References

[1] [http://en.wikipedia.org/wiki/Hash_table](http://en.wikipedia.org/wiki/Hash_table)  
[2] [http://www.amazon.com/Introduction-Algorithms-Thomas-H-Cormen/dp/0262033844/](http://www.amazon.com/Introduction-Algorithms-Thomas-H-Cormen/dp/0262033844/)  
[3] [http://en.wikipedia.org/wiki/Avalanche_effect](http://en.wikipedia.org/wiki/Avalanche_effect)  
[4] [http://blog.reverberate.org/2012/01/state-of-hash-functions-2012.html](http://blog.reverberate.org/2012/01/state-of-hash-functions-2012.html)  
[5] [http://www.strchr.com/hash_functions](http://www.strchr.com/hash_functions)  
[6] [http://programmers.stackexchange.com/questions/49550/which-hashing-algorithm-is-best-for-uniqueness-and-speed/145633#145633](http://programmers.stackexchange.com/questions/49550/which-hashing-algorithm-is-best-for-uniqueness-and-speed/145633#145633)  
[7] [http://blog.aggregateknowledge.com/2012/02/02/choosing-a-good-hash-function-part-3/](http://blog.aggregateknowledge.com/2012/02/02/choosing-a-good-hash-function-part-3/)  
[8] [https://sites.google.com/site/murmurhash/](https://sites.google.com/site/murmurhash/)  
[9] [http://google-opensource.blogspot.fr/2011/04/introducing-cityhash.html](http://google-opensource.blogspot.fr/2011/04/introducing-cityhash.html)  
[10] [http://incise.org/hash-table-benchmarks.html](http://incise.org/hash-table-benchmarks.html)  
[11] [http://preshing.com/20110603/hash-table-performance-tests](http://preshing.com/20110603/hash-table-performance-tests)  
[12] [http://attractivechaos.wordpress.com/2008/08/28/comparison-of-hash-table-libraries/](http://attractivechaos.wordpress.com/2008/08/28/comparison-of-hash-table-libraries/)  
[13] [http://attractivechaos.wordpress.com/2008/10/07/another-look-at-my-old-benchmark/](http://attractivechaos.wordpress.com/2008/10/07/another-look-at-my-old-benchmark/)  
[14] [http://blog.aggregateknowledge.com/2011/11/27/big-memory-part-3-5-google-sparsehash/](http://blog.aggregateknowledge.com/2011/11/27/big-memory-part-3-5-google-sparsehash/)  
[15] [http://gcc.gnu.org/](http://gcc.gnu.org/)  
[16] [https://code.google.com/p/sparsehash/](https://code.google.com/p/sparsehash/)  
[17] [http://fallabs.com/kyotocabinet/spex.html](http://fallabs.com/kyotocabinet/spex.html)  
[18] [http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1456.html](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1456.html)  
[19] [http://sparsehash.googlecode.com/svn/trunk/doc/implementation.html](http://sparsehash.googlecode.com/svn/trunk/doc/implementation.html)  
[20] [http://en.wikipedia.org/wiki/CPU_cache](http://en.wikipedia.org/wiki/CPU_cache)

Published in[Algorithms and Programming](https://codecapsule.com/category/algorithms/ "View all posts in Algorithms and Programming")[Implementing a Key-Value Store](https://codecapsule.com/category/implementing-a-key-value-store/ "View all posts in Implementing a Key-Value Store")

- [architecture](https://codecapsule.com/tag/architecture/ "View all posts tagged architecture")
- [C++](https://codecapsule.com/tag/c/ "View all posts tagged C++")
- [dense_hash_map](https://codecapsule.com/tag/dense_hash_map/ "View all posts tagged dense_hash_map")
- [diagram](https://codecapsule.com/tag/diagram/ "View all posts tagged diagram")
- [hash](https://codecapsule.com/tag/hash/ "View all posts tagged hash")
- [hash table](https://codecapsule.com/tag/hash-table/ "View all posts tagged hash table")
- [hashdb](https://codecapsule.com/tag/hashdb/ "View all posts tagged hashdb")
- [ikvs](https://codecapsule.com/tag/ikvs/ "View all posts tagged ikvs")
- [implementation](https://codecapsule.com/tag/implementation/ "View all posts tagged implementation")
- [key-value store](https://codecapsule.com/tag/key-value-store/ "View all posts tagged key-value store")
- [kyoto cabinet](https://codecapsule.com/tag/kyoto-cabinet/ "View all posts tagged kyoto cabinet")
- [organization](https://codecapsule.com/tag/organization/ "View all posts tagged organization")
- [sparsehash](https://codecapsule.com/tag/sparsehash/ "View all posts tagged sparsehash")
- [tr1](https://codecapsule.com/tag/tr1/ "View all posts tagged tr1")
- [unordered_map](https://codecapsule.com/tag/unordered_map/ "View all posts tagged unordered_map")

Previous Post[Estimated reading time](https://codecapsule.com/2013/04/27/estimated-reading-time/)

Next Post[Cuckoo Hashing](https://codecapsule.com/2013/07/20/cuckoo-hashing/)

## 13 Comments

1. ![Simon](https://secure.gravatar.com/avatar/fe2255e2b9f7dbef3f0a5028623baa3bbd0a70a2dd7823c167595d36c6b96502?s=48&d=identicon&r=g)Simon
    
    I had a great time reading your blog posts about hash table algorithms. I’m very interested to see how your own algorithm turns out! It would be great to be able to somehow subscribe to your blog in order to get automatic updates. There are a couple of aspects of hash tables that I wish you would investigate further: (1) Multi-threaded hash tables effects on performance due to locking and cache line contention, (2) Size of data and hash table overhead, e.g. redis apparently has an overhead of 96 bytes per key, and Aerospike 64 bytes per key, and (3) Performance regarding deleting and/or serialization, e.g. if I add 100 million keys it would be great to be able to serialize them by writing large blocks of memory rather than iterating and writing 100 million keys. — Simon
    
    June 1, 2013 [Reply](https://codecapsule.com/2013/05/13/implementing-a-key-value-store-part-5-hash-table-implementations/?replytocom=7306#respond)
    
    - ![Emmanuel Goossaert](https://secure.gravatar.com/avatar/fd1f104619cf7ca7bc1f582d36f58ff498dde5e1a7b0758f95a37453d98d5ada?s=48&d=identicon&r=g)[Emmanuel Goossaert](http://goossaert.com/)
        
        Hey Simon, thanks a lot for your comment! Those are great ideas, and below is what I can bring to the discussion for each of them. Also, I just added a plug-in to allow email subscription.
        
        (1) I have not investigated a lot regarding concurrency yet. I’d like to give a try to a lock-free solution, but in case locks are needed, then a major idea would be to design things so that the hash table can be locked locally. I imagine that there must be a lot of people doing this already, but I haven’t researched a lot about that.
        
        (2) The overhead is always a trade-off between space and functionality. When choosing a database solution, I always take a look at the average size of the data. Key-value stores are generally doing a great job when the data is in the order of a few hundred kilobytes. Below that, the overhead is larger than the data and not worth it. Indeed, storing 2 bytes of data with 64 bytes of overhead doesn’t make sense, and there might be better ways to access the data, based on its structure, that would avoid that. Above a couple of megabytes, RDBMS are doing a better job. This has been my experience so far, but of course it depends on the access patterns of each application. There is a discussion about this in the DynamoDB paper, Section 5 [1]. There are also solutions to large overheads, for instance the in-memory hash table sparse_hash_map from Sparsehash is using a sparsetable data structure internally, which allows an overhead of only 2 bits per entry. But to achieve this, a lot more time is spent shuffling things around to access the data, and therefore it is slower than more classic implementations with larger overheads.
        
        (3) Batching and deleting are both related to the more general problem of memory management, and I haven’t completely figured that out yet.
        
        The rule of thumb seems to call for avoiding as much as possible random writes on drives, and perform most of the writes sequentially (See [2], Section 13.2.1). This is what LevelDB is doing with its LSM tree data structure, and it’s performing very well. In addition to that, writing items in batch instead of individually allows to make more optimization regarding memory allocation, since more information about the data usage is known before the actual writing.
        
        I have read quite a bit about memory allocation for the kernel, and what is interesting is that the current algorithms — ptmalloc3, jemalloc and tcmalloc, all derived from dlmalloc — are optimized to be good general-purpose allocators. To ensure that, they are being tested against various real programs and memory allocation needs. See “Plots for Malloc-2.7.0” by Doug Lea [3] or the original jemalloc paper [4]. I have not been able to find any of that for key-value stores. All the key-value store benchmarks that I can find are doing reads and writes for both sequential and random access patterns, but no benchmark against custom applications. This means that we have no idea how those key-value stores are really performing with databases that are being used for months, and for which a whole bunch of random writes and deletes have been executed. This is why, in my opinion, memory management for key-value stores is a real low-hanging fruit. It would be great to create a database of test data based on the logs from the usage of key-value stores in large scale systems, and use that in benchmarks. My guess is that we would very probably find better memory management strategies that would improve performance dramatically.
        
        [1] [http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/decandia07dynamo.pdf](http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/decandia07dynamo.pdf)  
        [2] [http://www.aosabook.org/en/nosql.html](http://www.aosabook.org/en/nosql.html)  
        [3] [http://gee.cs.oswego.edu/dl/malloc-plots/index.html](http://gee.cs.oswego.edu/dl/malloc-plots/index.html)  
        [4] [http://people.freebsd.org/~jasone/jemalloc/bsdcan2006/jemalloc.pdf](http://people.freebsd.org/~jasone/jemalloc/bsdcan2006/jemalloc.pdf)