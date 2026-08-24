This is Part 3 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts.

In this article, I will walk through the architectures of Kyoto Cabinet and LevelDB, component by component. The goal, as stated in Part 2 of the IKVS series, is to get insights at how I should create the architecture my own key-value store by analyzing the architectures of existing key-value stores. This article will cover:

1. Intent and methodology of this architecture analysis  
2. Overview of the Components of a Key-Value Store  
2. Structural and conceptual analysis of Kyoto Cabinet and LevelDB  
    3.1 Create a map of the code with Doxygen  
    3.2 Overall architecture  
    3.3 Interface  
    3.4 Parametrization  
    3.5 String  
    3.6 Error Management  
    3.7 Memory Management  
    3.8 Data Storage  
3. Code review  
    4.1 Organization of declarations and definitions  
    4.2 Naming  
    4.3 Code duplication

![](https://i0.wp.com/codecapsule.com/wp-content/uploads/2012/12/kvstore_leveldb_kyotocabinet_small.jpg?resize=770%2C465 "kvstore_leveldb_kyotocabinet_small")

## 1. Intent and methodology of this architecture analysis

I was thinking whether I should do two separate articles, one for LevelDB and another one for Kyoto Cabinet, or a combined article with both. I believe that software architecture is a craft where decision making plays a very important role, as an architect needs to consider and choose among many alternatives for every part of a system. Solutions are never evaluated by themselves in isolation, but weighted against other solutions. The analysis of the architecture of a software system has value only if it is made in context, and compared to other architectures. For that reason, I will go through some of the main components encountered in a key-value store, and compare for each of them the solutions developed by existing key-value stores. I will use my own analyses for Kyoto Cabinet and LevelDB, but for other projects, I will use existing analyses. Here are the external analyses that I have chosen to use:

– BerkeleyDB, Chapter 4 in The Architecture of Open Source Applications, by Margo Seltzer and Keith Bostic (Seltzer being one of the two original authors of BerkeleyDB) [[1]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_1)  
– Memcached for dummies, by Tinou Bao [[2]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_2)  
– Memcached Internals [[3]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_3)  
– MongoDB Architecture, by Ricky Ho [[4]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_4)  
– Couchbase Architecture, by Ricky Ho [[5]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_5)  
– The Architecture of SQLite [[6]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_6)  
– Redis Documentation [[7]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_7)

## 2. Overview of the Components of a Key-Value Store

In spite of major differences in their internal architectures, key-value stores have very similar components. Below is a list of the major components encountered in most key-value stores, along with shorts descriptions of their utility.

**Interface:** The set of methods and classes exposed to the clients of a key-value store so they can interact it. This is also referred as the API. The minimum API for a key-value store must include the methods Get(), Put() and Delete().

**Parametrization:** The way that options are being set and passed to components across the whole system.

**Data Storage:** The interface used to access the memory where the data, i.e. keys and values, are stored. If the data must be _persisted_ on non-volatile storage such as hard drive or flash memory, then problems of _synchronization_ and _concurrency_ may arise.

**Data Structure:** The algorithms and methods being used to organize the data, and allow for efficient storage and retrieval. This is generally a hash table or B+ Tree. In the case of LevelDB, it is a Log-Structured Merge Tree. The choice of the data structure may depend on the internal structure of the data and the underlying data storage solution.

**Memory Management:** The algorithms and techniques being used to manage the memory used by the system. This is crucial as a data storage accessed with the wrong memory management technique can impact performance dramatically.

**Iteration:** The ways by which all the keys and values in a database can be enumerated and accessed sequentially. The solutions are mostly Iterators and Cursors.

**String:** The data structure used to represent and access strings of characters. This might seem like a detail, but for key-value stores, a great deal of time is being spent on passing and processing strings, and std::string from the STL might not be the best solution.

**Lock Management:** All the facilities related to the locking of concurrently accessed memory areas (with mutexes and semaphores), and the locking of files if the data storage is the file system. Also handles issues related to multithreading.

**Error Management:** The techniques used to intercept and handle errors encountered in the system.

**Logging:** The facilities that log the events happening in the system.

**Transaction Management:** Mechanism over a set of operations which ensures that all the operations are executed correctly, and in case of an error, that none of the operations is executed and the database is left unchanged.

**Compression**: The algorithms used to compress the data.

**Comparators:** Comparators provide ways to order two keys with regard to each other.

**Checksum:** The methods used to test and ensure the integrity of the data.

**Snapshot:** A Snapshot provides a read-only view of the entire database as it was when the snapshot was created.

**Partitioning**: Also referred to as Sharding, this consists in splitting the data set into multiple data storages, possibly distributed across multiple nodes on a network.

**Replication**: In order to ensure durability in case of system or hardware failures, some key-value stores allow for multiple copies of the data – or of partitions of the data – to be maintained simultaneously, preferably on multiple nodes.

**Testing Framework**: Framework being used to test the system, including unit and integration testing.

## 3. Structural and conceptual analysis of Kyoto Cabinet and LevelDB

The following analysis of LevelDB and Kyoto Cabinet will focus on the following components: _Parametrization, Data Storage, String and Error Management_. The components _Interface,_ _Data Structure, Memory Management, Logging and Testing Framework_ will be covered in future articles of the IKVS series. As for the rest of the components, I have no plans to cover them at the moment I am writing this article. Other systems, like relational databases, have other components such as Command Processor, Query Parser, and Planner/Optimizer, but all these are way beyond the scope of the IKVS series.

Before I start my analysis, please note that I consider both Kyoto Cabinet and LevelDB as great pieces of software, and I highly respect their authors. Even if I say bad things about their designs, keep in mind that their code is still awesome, and that I do not have accomplished what those guys did. This being said, you’ll find below my two cents about the code of Kyoto Cabinet and LevelDB.

### 3.1 Create a map of the code with Doxygen

In order to understand the architectures of Kyoto Cabinet and LevelDB, I had to dig into their source code. But I also used Doxygen, which is a very powerful tool to navigate through the hierarchies of modules and classes of an application. Doxygen is a documentation system for various programming languages, which can generate documentation directly form source code in the form of a report or HTML website. Most people add comments with a special format to their classes and methods, and then use Doxygen to generate a documentation that contains those special comments. However, Doxygen can also be used on code that does not contain any comment, and will generate an interface based the organization – files, namespaces, classes and methods – of the system.

You can get Doxygen on the official website [[8]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_8). After having installed Doxygen on your machine, just open a shell and go to the directory that contains all the source code that you want to analyze. Then type the following command to create a default configuration file:

$ doxygen -g

This will create a file called “Doxygen”. Open this file, and make sure that the following options are all set to “YES”: `EXTRACT_ALL, EXTRACT_PRIVATE, RECURSIVE, HAVE_DOT, CALL_GRAPH, CALLER_GRAPH`. These options will make sure that all entities are extracted from the code, even in sub-directories, and that call graphs are generated. Full descriptions of all the available options can be found in the online documentation of Doxygen [[9]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_9). To generate the documentation with the selected options, simply type:

$ doxygen Doxygen

The documentation will be generated in the “html” directory, and you can access it by opening the “index.html” file in any web browser. You can navigate through the code, see inheritance relationships between classes, and thanks to the graphs you can also see for every method which other methods it is calling.

## Join my email list

### 3.2 Overall architecture

Figure 3.1 and 3.2 represent the architecture of Kyoto Cabinet v1.2.76 and LevelDB 1.7.0, respectively. Classes are represented with the UML class diagram convention. Components are represented with corner-rounded rectangles, and black arrows represent the use of an entity by another. A black arrow from A to B means that A is using or accessing elements of B.

These diagrams try to represent the functional architecture as much as the structural architecture. For instance in Figure 3.1, many components are represented inside the HashDB class, because in the code those components are defined as part of the HashDB class.

In terms of internal component organization, there is no doubt that LevelDB is the big winner. The reason for this is that in Kyoto Cabinet the components for Iteration, Parametrization, Memory Management and Error Management are all defined as parts of the Core/Interface component, as shown in Figure 3.1. This creates a strong coupling of those components with the Core, and limits the modularity and future extensibility of the system. On the contrary, LevelDB is built in a very modular way, with only the Memory Management being part of the Core component.

[![](https://i0.wp.com/codecapsule.com/wp-content/uploads/2012/12/kvstore_kyotocabinet.jpg?resize=770%2C930 "kvstore_kyotocabinet")](https://i0.wp.com/codecapsule.com/wp-content/uploads/2012/12/kvstore_kyotocabinet.jpg)

Figure 3.1

[![](https://i0.wp.com/codecapsule.com/wp-content/uploads/2012/12/kvstore_leveldb.jpg?resize=770%2C900 "kvstore_leveldb")](https://i0.wp.com/codecapsule.com/wp-content/uploads/2012/12/kvstore_leveldb.jpg)

Figure 3.2

### 3.3 Interface

The interface of the HashDB class of Kyoto Cabinet is exposing more than 50 methods, versus only 15 for the DBImpl class of LevelDB (and four of these 15 are for test purposes). This is a direct consequence of the strong coupling taking place in the Core/Interface component for Kyoto Cabinet, such as the definition of the Parametrization module inside the Core.

API design will be discussed in more details in a future article of the IKVS series.

### 3.4 Parametrization

In Kyoto Cabinet, the parameters are tuned by calling methods of the HashDB class. There are 15 methods like that, all with the prefix “tune_”.

In LevelDB, the parameters are defined into specialized objects. “Options” for the general parameters, and “ReadOptions” and “WriteOptions” for parameters of the Get() and Put() methods respectively, as represented in Figure 3.2. This decoupling enables a better extensibility of the options, without messing with the public interface of the Core like this is the case with Kyoto Cabinet.

### 3.5 String

In key-value stores, there is a lot of string processing going on. Strings are being iterated, hashed, compressed, passed, and returned. Therefore, a clever implementation of String is very important, as tiny savings in objects used on a large scale can have a dramatic impact globally.

LevelDB is using a specialized class called “Slice” [[10]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_10). A Slice holds a byte array along with the size that array. This allows to know the size of the string in time O(1), ~~unlike std::string which would take O(n)~~ unlike strlen() on C strings which would take O(n). Note that in C++, size() for std::string is also O(1). Having the size stored separately also allows for the ‘\0’ character to be stored, which means the keys and values can be real byte array and not just null-terminated strings. Finally and more importantly, the Slice class handles the copy by making a shallow copy, not a deep copy. Meaning, it simply copies the pointer to the byte array, and doesn’t make a full copy of the byte array like std::string. This avoids copying potentially very large keys and values.

Like LevelDB, Redis is using its own data structure to represent strings. The goal expressed is also to avoid an O(n) operation to retrieve the size of the string [[11]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_11).

Kyoto Cabinet is using std::string for its strings.

My opinion is that an implementation of String, adapted to the requirements of the key-value stores, is absolutely necessary. Why spend time copying strings and allocating memory if it can be avoided?

### 3.6 Error Management

In all the C++ source code that I have been looking at for key-value stores, I have not seen a single use of exceptions being used as the global error management system. In Kyoto Cabinet, the threading component in the kcthread.cc file is using the exceptions, but I think that this choice is more related to the handling of threads than a general architectural choice. Exceptions are dangerous, and should be avoided whenever possible.

BerkeleyDB has a nice C-style way to handle errors. Error message and error codes are all centralized in one file. All functions that return error codes have a integer local variable named “ret”, which is filled while processing and returned at the end. This approach is rolled out in all files, and in all modules: very polished, normalized error management. In some functions, a few forward jumping gotos are used, a technique widely used in serious C-based system such as the Linux kernel [[12]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_12). Even though this error management approach is very clear and clean, a C-style error management would not make much sense in a C++ application.

In Kyoto Cabinet, one Error object is stored in every database object such as HashDB. In the database classes, methods are calling set_error() to set the Error object in case an error occurs, and return true or false in very a C-style way. No local variable returned at the very end of the methods like in BerkeleyDB, return statements are placed wherever the errors occur.

LevelDB is not using exceptions at all, but a special class called Status. This class holds both an error value and an error message. This object is returned by all methods so that the error status can be either treated on the spot or passed to other methods higher up in the calling stack. This Status class is also implemented in a very clever way, as the error code is stored inside the string itself. My understanding of this choice is that most of the time, the methods will return a Status of “OK”, to say that no error was encountered. In that case, the message string is NULL, and the occurrence of the Status object is very light. Had the authors of LevelDB chosen to have one additional attribute to store the error code, this error code would have had to be filled even in the case of a Status of “OK”, which would have meant more space used on every method call. All components are using this Status class, and there is no need to go through a centralized method as with Kyoto Cabinet, as shown in Figure 3.1 and 3.2.

Of all the error management solutions presented above, I personally prefer the solution used in LevelDB. This solution avoids the use of exceptions, it is not a simple C-style error management which is too limited in my opinion, and it prevents any unnecessary coupling with the Core component like it is the case with Kyoto Cabinet.

### 3.7 Memory Management

Both Kyoto Cabinet and LevelDB have the memory management defined inside the Core component. For Kyoto Cabinet, the memory management consists of keeping track of the block of contiguous free memory in the database file on disk, and selecting a block of adequate size whenever an item is being stored. The file itself is just memory mapped with the mmap() function. Note that MongoDB too is using a memory mapped file [[13]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_13).

For LevelDB, which implements a Log-Structured Merge Tree, there are no gaps of free space in the file as it is the case with hash tables stored on disk. The memory management consists in compacting the log files whenever they exceed a certain size [[14]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_14).

Other key-value stores, such as Redis, use memory allocation with malloc() — in the case of Redis, the memory allocation algorithm is not the one provided by the operating system like dlmalloc or ptmalloc3, but jemalloc [[15]](https://codecapsule.com/2012/12/30/implementing-a-key-value-store-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb/#ref_15).

Memory management will be described in details in a later article of the IKVS series.

### 3.8 Data Storage

Kyoto Cabinet, LevelDB, BerkeleyDB, MongoDB and Redis are using the file system to store the data. Memcached, on the contrary, is storing the data in memory (RAM).

Data storage will be described in details in a later article of the IKVS series.

## 4. Code review

This section is a quick code review of Kyoto Cabinet and LevelDB. It is not thorough, and only contains elements that I judged remarkable when I was reading the source code.

### 4.1 Organization of declarations and definitions

If the code is normally organized in LevelDB, with the declarations in the .h header files and the definitions in the .cc implementation files, I have found something shocking in Kyoto Cabinet. Indeed, for many classes, the .cc files do not contain any definition, and the methods are all being defined directly from the headers. In other files, some methods are defined in the .h and some others in the .cc files. While I understand that there might be a reason behind this choice, I still find that not following such a respected convention in a C++ application is fundamentally wrong. This is wrong because it makes me wonder with it is like that, and it makes me look into two different files for the implementations, after years of C++ and looking into just one type of files, the .cc files.

### 4.2 Naming

First of all, the code of Kyoto Cabinet is a significant improvement compared to the code of Tokyo Cabinet. The overall architecture and naming conventions have been greatly improved. Nevertheless, I still find many of the names in Kyoto Cabinet to be very cryptic, with attribute and method names such as `embcomp, trhard, fmtver(), fpow()`. It feels like some C code got lost into some C++ code. On the other hand, the naming in LevelDB is very clear, except maybe for some temporary variables with names such as `mem, imm, and in`, but that’s very minimal and the code remains extremely readable.

### 4.3 Code duplication

I have seen quite a bit of code duplication in Kyoto Cabinet. The code that is used to defragment a file is repeated at least three times, and all the methods that require a branching between Unix and Windows versions all show a great deal of duplication. I have not found any significant piece of duplicated code in LevelDB. I am sure there must be some too, but I would have to dig a lot deeper to find it, proof that it is a lesser problem in LevelDB than it is in Kyoto Cabinet.

## Join my email list

## Translations

This article was translated to [Simplified Chinese](http://blog.xiongduo.cn/IT-Trans/ikvs-part-3-comparative-analysis-of-the-architectures-of-kyoto-cabinet-and-leveldb.html) by Xiong Duo.

## References

[1] [http://www.aosabook.org/en/bdb.html](http://www.aosabook.org/en/bdb.html)  
[2] [http://work.tinou.com/2011/04/memcached-for-dummies.html](http://work.tinou.com/2011/04/memcached-for-dummies.html)  
[3] [http://code.google.com/p/memcached/wiki/NewUserInternals](http://code.google.com/p/memcached/wiki/NewUserInternals)  
[4] [http://horicky.blogspot.com/2012/04/mongodb-architecture.html](http://horicky.blogspot.com/2012/04/mongodb-architecture.html)  
[5] [http://horicky.blogspot.com/2012/07/couchbase-architecture.html](http://horicky.blogspot.com/2012/07/couchbase-architecture.html)  
[6] [http://www.sqlite.org/arch.html](http://www.sqlite.org/arch.html)  
[7] [http://redis.io/documentation](http://redis.io/documentation)  
[8] [http://doxygen.org](http://doxygen.org/)  
[9] [http://www.stack.nl/~dimitri/doxygen/config.html](http://www.stack.nl/~dimitri/doxygen/config.html)  
[10] [http://leveldb.googlecode.com/svn/trunk/doc/index.html](http://leveldb.googlecode.com/svn/trunk/doc/index.html)  
[11] [http://redis.io/topics/internals-sds](http://redis.io/topics/internals-sds)  
[12] [http://news.ycombinator.com/item?id=3883310](http://news.ycombinator.com/item?id=3883310)  
[13] [http://www.briancarpio.com/2012/05/03/mongodb-memory-management/](http://www.briancarpio.com/2012/05/03/mongodb-memory-management/)  
[14] [http://leveldb.googlecode.com/svn/trunk/doc/impl.html](http://leveldb.googlecode.com/svn/trunk/doc/impl.html)  
[15] [http://oldblog.antirez.com/post/everything-about-redis-24.html](http://oldblog.antirez.com/post/everything-about-redis-24.html)