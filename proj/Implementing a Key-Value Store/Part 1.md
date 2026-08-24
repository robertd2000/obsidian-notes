This is Part 1 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts.

In this article, I will start with a short description of what key-value stores are. Then, I will explain the reasons behind this project, and finally I will expose the main goals for the key-value store that I am planning to implement. Here is the list of the things I will cover in this article:

1. A quick overview of key-value stores  
2. Key-value stores versus relational databases  
3. Why implement a key-value store  
4. The plan

## 1. A quick overview of key-value stores

This section is intended to be a very short introduction to key-value stores, as many more detailed articles have been written already. I have selected a few ones that you will find in the References section at the bottom of this article.

Key-value stores are one of the simplest forms of database. Almost all programming languages come with in-memory key-value stores. The map container from the C++ STL is a key-value store, just like the HashMap of Java, and the dictionary type in Python. Key-value stores generally share the following interface:

**Get( key ):** Get some data previously saved under the identifier “key”, or fail if no data was stored for “key”.

**Set( key, value ):** Store the “value” in memory under the identifier “key”, so we can access it later referencing the same “key”. If some data was already present under the “key”, this data will be replaced.

**Delete( key ):** Delete the data that was stored under the “key”.

Most underlying implementations are using either hash tables or some kind of self-balancing trees, like B-Trees or Red-black trees. Sometimes, the data is too big to fit in memory, or the data must be persisted in case the system crashes for any reason. In that case, using the file system becomes mandatory.

Key-value stores are part of the NoSQL movement, which regroup all the database systems that do no make use of all the concepts coined by relational databases. The [Wikipedia article on NoSQL](http://en.wikipedia.org/wiki/NoSQL) sums up very well what are the main characteristics of those databases:

- Do not use the SQL query language
- May not provide full support of the [ACID paradigm](http://en.wikipedia.org/wiki/ACID) (atomicity, consistency, isolation, durability)
- May offer a distributed, fault-tolerant architecture.

## 2. Key-value stores and relational databases

Unlike relational databases, key-value stores have no knowledge of the data in the values, and do not have any schema like in MySQL or PostgreSQL. This also means that it is impossible to query only part of the data by doing any kind of filtering, as it can be done in SQL with the WHERE clause. If you do not know where to look for, you will have to iterate over all the keys, get their corresponding values, apply whatever filtering that you need on those values, and keep only the ones you need. This can be very computationally intensive, which implies that for full performance can only be attained in the cases where the keys are known, otherwise key-value stores turn up to be simply inadequate (Note: some key-value stores can store structured data and have fields indexed).

Therefore, even if key-value stores often outperform relational database systems by several orders of magnitude in terms of sheer access speed, the requirement to know the keys restricts the possible applications.

## Join my email list

## 3. Why implement a key-value store

I am starting this project as a way to refresh my knowledge of some fundamentals of hardcore back-end engineering. Reading books and Wikipedia articles is boring and exempt of practice, so I thought it would be better to get down to business and get my hands dirty with some code. I was looking for a project that would allow me to review:

- The C++ programming language
- Object-oriented design
- Algorithmics and data structures
- Memory management
- Concurrency control with multi-processors or multi-threading
- Networking with a server/client model
- I/O problems with disk access and use of the file system

A key-value store that is using the file system for permanent storage and which offers a networking interface would cover the whole range of topics listed above. This is just the perfect project that deals with all domains of back-end engineering.

But let’s be realistic for a minute. There are already a lot of key-value stores out there, some of which have been implemented by really smart guys, and that are already in use in production in very big companies. These include names like Redis, MongoDB, memcached, BerkeleyDB, Kyoto Cabinet and LevelDB.

In addition to that, there appears to be a trend around key-value stores these days. It seems like everybody has one and wants to show how awesome and fast their key-value stores are. This issue is also described in [Leonard Lin’s blog article on key-value stores](http://randomfoo.net/2009/04/20/some-notes-on-distributed-key-stores). The majority of these projects at the moment are not mature and really not ready for production, but still, people like to show them off. It is very common to find blog articles or conference slides showing a comparison of the performance of some obscure key-value stores. Those charts are generally worth nothing, and only a test on your own hardware and with your own data and application will tell you which ones of these key-value stores are the most adapted to solve your problem. This is because performance depends on:

- The hardware
- The file system being used
- The actual application and order in which the keys are being accessed ([locality of reference](http://en.wikipedia.org/wiki/Locality_of_reference))
- The data set, and in particular the lengths of keys and values and the possibility for key collisions when hash tables are used

Therefore, coding a key-value store that will have an impact is going to be relatively hard, because it is very likely to end up unnoticed under the weight of the best key-value stores already available, or to simply drown in the ocean of half-baked side projects that nobody cares about.

In order to be different, this project cannot run for speed like everybody else, and must aim at filling a gap left by the solutions already available. Here are a few ways I see to have a key-value store project stand out from the crowd:

- Adapt to a specific data representation (ex: graphs, geographic data, etc.)
- Adapt to a specific operation (ex: performing very well for reads only, or writes only, etc.)
- Adapt to a specific issue (ex: automatic parameter tuning, as many key-value stores have many options, and it’s sometimes a mess to find the best ones)
- Offer more data access options. For instance in LevelDB, data can be accessed forward or backward, with iterators, and it has sorting on the keys. Not all key-value stores can do that.
- Make the implementation more accessible: right now, very few key-value stores have their code fully documented. If you need to get a project running quickly and you have to customize a key-value store for that, one that has code which seems really accessible will be a possible choice, even if it’s not a very well-known project. The fact that one can understand the code and therefore trust it will compensate for that.
- Specific applications. Here is an example of a real problem: many web crawling frameworks (web spiders) have a poor interface to manage the URLs they have to crawl, and this is often left to the client to implement the logic using a key-value store. All web crawling frameworks could benefit from a unified key-value store optimized for URL management.

## 4. The plan

The goal of this project is to develop a lightweight key-value store in understandable C++ code. As a matter of fact, I am planning to follow the [Google C++ style guide](http://google-styleguide.googlecode.com/svn/trunk/cppguide.xml) for this project. I will use a hash table for the underlying data structure, the data will be persistent on disk, and a network interface will also be implemented. I will not run for absolute speed, but for conciseness and clarity in both design and implementation. I will also try the best I can to minimize the memory footprint of the database file on disk.

I do not want to re-invent the wheel, so I will start by looking around for key-value store projects in C or C++, and select those that stand out due to their quality. I will proceed to study their architecture and code, and draw inspiration from them for my own implementation. As back-end engineering is one of my core skills, I already have most of the knowledge needed for this project, but I know that I am going to learn a lot of new things too, and that’s what makes it interesting to me. I am also very happy to be documenting this whole thing. In the past, I have enjoyed hardcore technical blogs such as the ones of [Alexander Sandler](http://www.alexonlinux.com/) and [Gustavo Duarte](http://duartes.org/gustavo/blog/best-of), and I wanted to contribute with something useful myself, as best as I can.

The result of my research and my work on key-value stores will be documented in this blog article series. Do not try to use the dates of the articles to estimate the time it takes to implement a key-value store: the articles may be published with significant delays compared to the research and actions they describe.

In Part 2, I will search for top key-value stores projects and explain why I choose some of them as models and not others. For other articles, you can refer to the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) of the IKVS article series.

You will find below in the References section a few articles and book chapters to learn more about key-value stores. Before reading Part 2, I strongly recommend you to read at least [The NoSQL Ecosystem](http://www.aosabook.org/en/nosql.html) and [Key Value Stores: A Practical Overview](http://blog.marc-seeger.de/2009/09/21/key-value-stores-a-practical-overview/).

## Join my email list

## Translations

This article was translated to [Simplified Chinese](http://blog.xiongduo.cn/IT-Trans/ikvs-part-1-what-are-key-value-stores-and-why-implement-one.html) by Xiong Duo.

## References

[The NoSQL Ecosystem](http://www.aosabook.org/en/nosql.html), from the book “Architecture of Open Source Applications, Volume 1”  
[NoSQL Patterns](http://horicky.blogspot.com.es/2009/11/nosql-patterns.html), by Ricky Ho  
[NoSQL Databases](http://nosql-database.org/), referencing all the NoSQL databases that matter at the moment  
[NoSQL Data Modeling Techniques](http://highlyscalable.wordpress.com/2012/03/01/nosql-data-modeling-techniques/), by Ilya Katsov  
[NoSQL databases benchmark: Cassandra, HBase, MongoDB, Riak](http://www.networkworld.com/cgi-bin/mailto/x.cgi?pagetosend=/news/tech/2012/102212-nosql-263595.html), and the discussion on [Hacker News](http://news.ycombinator.com/item?id=4733212)  
[Wikipedia article on NoSQL](http://en.wikipedia.org/wiki/NoSQL)  
[Wikipedia article on the ACID paradigm  
](http://en.wikipedia.org/wiki/ACID)[Key Value Stores: A Practical Overview](http://blog.marc-seeger.de/2009/09/21/key-value-stores-a-practical-overview/), by Marc Seeger  
[Some Notes on Distributed Key Stores](http://randomfoo.net/2009/04/20/some-notes-on-distributed-key-stores), by Leonard Lin