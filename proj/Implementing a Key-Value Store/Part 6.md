This is Part 6 of the IKVS series, “Implementing a Key-Value Store”. You can also check the [Table of Contents](http://codecapsule.com/2012/11/07/ikvs-implementing-a-key-value-store-table-of-contents/) for other parts.

In this article, I will compare several open-addressing hash tables: Linear Probing, Hopscotch hashing, and Robin Hood hashing. I have already done some work on this topic, and in this article I want to gather data for more metrics in order to decide which hash table I will use for my key-value store.

The result section also contains an interesting observation about the maximum DIB for Robin Hood hashing, which originated from Kristofer Karlsson, a software engineer at Spotify and the author of the key-value store Sparkey.

This article will cover:

1. Open-addressing hash tables  
2. Metrics  
3. Experimental Protocol  
4. Results and Discussion  
5. Conclusion  
6. References

## 1. Open-addressing hash tables

I have settled on open-addressing hash tables as the data structure for my key-value store, because such algorithms minimize the number of I/O operations to access entries, which is ideal when the data is persisted to a secondary storage such as HDD or SSD. However, one of the main drawbacks of hash tables compared to other data structures such as trees is that they make it difficult to access the entries sorted by keys. I am aware of this is a trade-off, and I make the choice to pursue with open-addressing hash tables anyway.

There are several open addressing algorithms that are great candidates — Linear Probing, Cuckoo hashing, Hopscotch hashing, and Robin Hood hashing — and which I have described in details in previous articles [[1, 3, 6, 7]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref). I have chosen not to pursue my tests any further with Cuckoo hashing, because even if it is an interesting approach, it forces data to be read from non-contiguous memory locations, which is not cache-friendly and incurs additional I/O accesses when persisted to a secondary storage. The rest of this article will therefore only consider Linear Probing, Hopscotch hashing, and Robin Hood hashing.

## 2. Metrics

In my previous articles about hashing [[1, 3, 6, 7]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref), I was only using one metric to assess the performance of the hash tables: the distance to initial bucket (DIB). Since then I have collected data for more metrics to get a better understanding of the behavior of hash tables, and I am presenting those metrics in this section. In addition, Figure 1 below gives a quick visual representation of the DIB, DFB and DMB.

### 2.1 DIB: Distance to Initial Bucket

The distance to the initial bucket (DIB) for a given entry is the distance between the bucket where an entry is stored and its initial bucket, which is the bucket to which its key was initially hashed. This metric is the most important of all because it shows how much work is required to perform an entry retrieval, which is used by all the operations of a hash table: exists(), get(), put() and remove(). Keeping that metric as low as possible is crucial for achieving high performance. Note that the original Robin Hood paper [[8]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref) is calling this metric the “_Probe Sequence Length_” (PSL).

### 2.2 DFB: Distance to Free Bucket

The distance to the free bucket (DFB) for a given entry is the distance that the put() method has to scan from the initial bucket until it can find a free bucket (i.e., an empty bucket).

### 2.3 DMB: Distance to Missing Bucket

The distance to the missing bucket (DMB) is the distance that the get() method has to scan from the initial bucket until it can conclude that an entry is not in the hash table.

### 2.4 DSB: Distance to Shift Bucket

The distance to the shift bucket (DSB) measures the size of the shift during a deletion for Robin Hood hashing with backward shift deletion (RH-backshift). During a deletion, the index of the entry to delete is retrieved, and from there the position of the bucket where the shift can stop is determined (shift bucket). The DSB is therefore calculated as the distance between the index of the entry to delete and the index of the shift bucket. In this aspect, the DSB differs from the DIB, DMB and DFB, which are computed from the initial bucket.

![kvstore-part6-metrics](https://i0.wp.com/codecapsule.com/wp-content/uploads/2014/05/kvstore-part6-metrics.png?resize=530%2C350)

Figure 1: Visual representation of the DIB, DFB and DMB

### 2.5 Number of bucket swaps

Hopscotch hashing and Robin Hood hashing are both using a reordering scheme, which swaps buckets during insertions. Each swap incurs a write operation in the bucket array, and thus it is important to look at the number of swaps performed to see if they impact performance.

### 2.6 Aligned DIB, DFB, DMB, and DSB

Lower-level components have to deal with memory alignment. In order to get a better idea as to how much memory will actually be accessed for each operation, I have computed the aligned versions of all the distance metrics discussed above.

For example, if an entry is stored in the bucket just after its initial bucket — with indexes (i) and (i+1) — then the DIB for that entry is (i+1)-(i) = 1. Assuming 16-byte buckets, the difference between the offsets of these two indexes is 16 bytes. But to perform any operation on both buckets, both need to be read, and therefore 32 bytes of memory in total need to be accessed. If these two buckets happen to be at the edge of two 64-byte cache lines, with the initial bucket at the end of a cache line and the storage bucket at the beginning of the next one, then 128 bytes of memory in total need to be accessed.

Because I wanted to know the amount of memory accessed, I am not just aligning the distance metric data points, I am really calculating how much aligned memory is accessed. In that sense, it is a bit of a lie to call those metrics “aligned distances”, because they do not represent distances, but the amount of aligned memory required to access the data.

### 2.7 Other metrics of interest

There are two other metrics that would be interesting to look at, but that I have decided to not cover in this article.

The first metric of interest is the number of exact collisions, i.e. the number of keys for which the hash functions outputs the exact same bits. For on-disk hash tables, it makes sense to store the hashes of the keys in the buckets. That way during a lookup, a quick comparison on the hashes can be done to only retrieve the full key data when the hashes match. If there are many exact collisions, then many unsuccessful retrieval of key data will be performed, wasting a lot of I/O. This metric depends only on the hash function, not on the hash table algorithm, which is why it is not included in this article.

The second metric of interest is the number of cache misses. Open-addressing hash tables are expected to perform well due to their cache-friendliness, although it is good to keep an eye on the actual number of cache misses in the CPU to verify that the algorithms deliver what they promise. I have decided not to include this metric here because it is mostly related to CPU architecture, thus it is something that will be best monitored directly on the end system where the hash tables will be running.

## Join my email list

## 3. Experimental protocol

### 3.1 Test cases

I have used the same test cases that I used in my previous articles about Robin Hood hashing, the “loading”, “batch”, and “ripple” test cases. Below are full descriptions of what those test cases are doing:

**The “loading” test case:**  
– Insert entries in the hash table until it full, up to a load factor of 0.98

– Measure statistics at every 0.02 increment of the load factor

**The “batch” test case:**  
Uses two parameters, Load Factor Max (LFM) and Load Factor Remove (LFR)  
– Insert entries in the table up to LFM (with a table of 10k entries and LFM=0.8, 8k entries would be inserted)  
– Do the following operations over 50 iterations (for 1 <= i <= 50):

- Remove at once LFR entries of the total capacity (with a table of 10k entries and LFR=0.1, 1k entries would be removed at once)
- Insert at once LFR entries of the total capacity (with a table of 10k entries and LFR=0.1, 1k entries would be inserted at once)
- Measure statistics at the i-th iteration

**The “ripple” test case:**  
Uses two parameters, Load Factor Max (LFM) and Load Factor Remove (LFR)  
– Insert entries in the table up to LFM (with a table of 10k entries and LFM=0.8, 8k entries would be inserted)  
– Do the following operations over 50 iterations (for 1 <= i <= 50):

- Remove a single entry, and immediately insert a single entry. Do this for LFR of the total capacity (with a table of 10k entries and LFR=0.1, a single entry would be removed, then a single entry would be inserted, and this, 1000 times)
- Measure statistics at the i-th iteration

### 3.2 Algorithm comparison

For each algorithm that I am studying, I have run simulations of the test cases described above with the following parameters:

- “batch” test case, with LFM=0.8 and LFR=0.1
- “batch” test case, with LFM=0.8 and LFR=0.8
- “ripple” test case, with LFM=0.8 and LFR=0.1
- “loading” test case

The keys for the entries are generated using the random function from the C++ standard library. For each test case, 50 **instances** have been run. This means that for the “batch” and “ripple” test cases, I have run 50 times 50 iterations, then I have averaged the 50 values corresponding to each iteration. For the “loading” test case, I have also run 50 instances, but there is only one iteration, which is why the x-axis in the graphs is not “_Iterations_” but “_Load factor_“. For each instance, the same seed value was used across all algorithms, in such a way that the k-th instance for a test case over any algorithm was run using the same random keys as for the other algorithms, making it a fair comparison. Finally for each metric, I have calculated the following statistical aggregates: mean, median, 95th percentile, maximum, and variance.

It is important to notice that the metrics are gathered at different moments for each test case. For example for the “loading” test case, each metric is computed after an increment of 2% and therefore over the 2% of the population that was the most recently inserted into the table. For the batch with LFM=0.8 and LFR=0.8, the metrics are computed after the table has been cleared and filled up to a load factor of 0.8, thus they represent the entire population of entries in that hash table.

### 3.3 Robin Hood hashing table size comparison

To determine if the table size has an effect on the performance of Robin Hood hashing, I have run the “loading” test case for different sizes of the hash table: 10k, 100k, 1M and 10M. Each table size is run over 10 iterations.

## 4. Results and Discussion

### 4.1 Raw data

The source code for all the hash tables presented here is available in a public repository [[10]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref).

The raw data from the simulations that I have run can be downloaded from the data repository [[11]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref).

From this data, I have plotted a significant amount of graphs, taking into account the various test cases and metrics discussed above, along with the multiple statistical aggregates. All the graphs are available from the data repository [[11]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref). For the sake of length, I am not presenting all the graphs in this article, only the ones that I think are the most meaningful.

### 4.2 Robin Hood hashing has the best DIB and DMB, and an acceptable DFB

In a hash table, all the operations — get(), put(), exists() and remove() — need to determine the position of the entry on which they are operating, which efficiency is measured by the DIB. For the 95th percentile of aligned DIB, as shown in Figures 2(a) and 2(d) below, Robin Hood hashing with backward shift deletion (RH-backshift) tops at 256 bytes, whereas Linear Probing and both Hopscotch hashing algorithms go up to 512 bytes and 1 KB respectively. Having a low DIB gives RH-backshift a clear advantage over other algorithms.

![aligned dib_perc95](https://i0.wp.com/codecapsule.com/wp-content/uploads/2014/05/aligned-dib_perc95.png?resize=704%2C651)

Figure 2: 95th percentile of the aligned distance to initial bucket (DIB) for Linear Probing, Hopscotch Hashing, and Robin Hood hashing, over four test cases

For the distance to missing bucket (DMB), RH-backshift clearly outperforms all the other algorithms and minimizes the cost of looking up keys that are absent from the hash table, and this even if the hash table is filled up to a high load factor, here 0.8. In Figure 3 below, the mean of the aligned DMB shows that the cost of looking up absent keys will require to scan at most 32 bytes for RH-backshift. Compared to the bitmap Hopscotch or the Linear Probing which require 512 bytes and 16 KB respectively, this is impressive.

![aligned dmb_mean](https://i0.wp.com/codecapsule.com/wp-content/uploads/2014/05/aligned-dmb_mean.png?resize=704%2C651)

Figure 3: Mean of the aligned distance to missing bucket (DMB) for Linear Probing, Hopscotch Hashing, and Robin Hood hashing, over four test cases

The distance to free bucket (DFB) is the metric for which RH-backshift isn’t so shinny. The mean and median DFB of RH-backshift are higher than the mean and median for Hopscotch and Linear Probing. However, looking at the aligned DFB, RH-backshift shows acceptable performance with the mean of the aligned DFB at around 64 bytes as seen in Figure 4, thus within the critical limitation of a 64-byte cache line size.

![aligned dfb_mean](https://i0.wp.com/codecapsule.com/wp-content/uploads/2014/05/aligned-dfb_mean.png?resize=704%2C651)

Figure 4: Mean of the aligned distance to free bucket (DFB) for Linear Probing, Hopscotch Hashing, and Robin Hood hashing, over four test cases

### 4.3 In Robin Hood hashing, the maximum DIB increases with the table size

Kristofer Karlsson, a software engineer at Spotify, has a lot of experience with Robin Hood hashing. He is the author of Sparkey, an on-disk key-value store that he wrote for Spotify and which makes use of Robin Hood hashing. The code of Sparkey has been made open source and is available in its public repository [[12]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref).

In an email discussion, Kristofer pointed out that the maximum DIB was growing with the size of the table. At first I found that result a bit strange, and since I only had been playing with tables of relatively small sizes — 10k to 100k items — I decided to run some tests with my code on larger tables, up to 50M entries. The results I got were showing a growth of the maximum DIB as a function of the table size, which confirmed what Kristofer had observed. Reading over Celis’s thesis again [[8]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref), I realized that Celis did mention the growth pattern of the maximum DIB, on page 46:

> For a fixed load factor, the longest probe sequence length grows slowly when the table size is increased […] the growth is at most logarithmic in the table size.

With this new data and the indication from Celis’s thesis, Kristofer and I were able to confirm that the maximum DIB was growing logarithmically with the table size, and that this was indeed the expected behavior.

This growth in the maximum DIB can be observed in Figure 5(d), which shows the “loading” test case for different table sizes of Robin Hood hashing. The same growth behavior can also be observed for the maximum DFB, DMB, and DSB. Nevertheless, it is interesting to notice that the mean, median and 95th percentile of the DIB are the same for any table size, as shown by Figure 5(a), 5(b) and 5(c). Even though the maximum is growing with the size of the table, the average performance is independent from it, and remains good for any table size.

![dib](https://i0.wp.com/codecapsule.com/wp-content/uploads/2014/05/dib.png?resize=704%2C795)

Figure 5: All statistics for the distance to the initial bucket (DIB) of Robin Hood hashing with backward shift deletion, with different table sizes over the “loading” test case

### 4.4 Robin Hood hashing has an acceptable rate of swapped buckets during insertions

My biggest concern with RH-backshift is the number of buckets swapped during insertion, which can rise up to 80 at peak, as it can be seen in Figure 6(a). Buckets are going to be swapped starting from the initial bucket until a free bucket is found. As mentioned above, the aligned DFB for RH-backshift is at around 64 bytes, the size of a cache line. During an insert, the cache line would need to be updated anyway, therefore as the aligned DFB is within 64 bytes, the swaps will be written to the cache line along with the update made for the insert, not causing any additional work.

![swap_maximum](https://i0.wp.com/codecapsule.com/wp-content/uploads/2014/05/swap_maximum.png?resize=704%2C651)

Figure 6: 95th percentile of the swaps for Linear Probing, Hopscotch Hashing, and Robin Hood hashing, over four test cases

### 4.5 Hopscotch hashing uses bucket swaps only at high load factors

Following my intuition, I would have expected bucket swaps to occur frequently in a Hopscotch hash table, but looking at the swap metric, it is not the case. Indeed, if for Robin Hood hashing the swaps occur at any load factor, for Hopscotch hashing they only start occurring in large numbers starting at load factors of 0.7. I suspected that this could be something due my own implementation, therefore to clear up my doubts I decided to use the code made by the authors of the Hopscotch hashing paper [[5]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref), and I dumped the number of swaps occurring in their bitmap neighbourhood implementation with the following parameters:

```
./test_intel32 bhop 1 2 100 90 102400 1 1
```

This gave me the following output:

```
ConcurrentHashMap Benchmark
---------------------------
numOfThreads:   2
Algorithm Name: bhop
NumProcessors:  1
testNo:         1
insert ops:     50
del ops:        50
constain ops:   0
loadFactor:     89
initialCount:   131072
throughputTime: 1
initialCapacity:   131072

START create random numbers.
END   creating random numbers.

START fill table. (117964)
BHOP Num elm: 117964
END   fill table.

START creating threads.
END   creating threads.

START threads.
END   threads.

STARTING test.
ENDING test.
```

I compared this to my own implementation [[10]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref) using the following parameters:

```
./hashmap --algo bitmap --num_buckets 140000 --testcase batch --load_factor_max 0.89 --load_factor_step 0.89
```

Looking at the distributions of the swap metric for both implementations, it appeared that they are very similar, which gives me confidence that my implementation is valid, at least as valid as the one of the original authors of the Hopscotch algorithm. The swap distributions are available in the data repository [[12]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref).

Hopscotch hashing will perform swaps only when it is unable to fill up the neighborhood of a bucket, around the time when the “neighborhood clustering” [[3]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref) effect starts to appear and renders the hash table unusable unless it is resized.

### 4.6 Robin Hood hashing has an acceptable DSB

Another concern for RH-backshift is the number of writes necessary to shift buckets backward after a bucket is removed, which is measured by the distance to the shift bucket (DSB). Note that if the DIB, DMB and DFB are computed starting from the initial bucket, the DSB is computed from the bucket where the removed entry used to be stored, because this is the location up to which data will be written.

Even if the 95th percentile and the maximum of the DSB can reach high values, looking at the mean of the aligned DSB, it appears that they stay at around 64 bytes for the test cases having the heaviest workloads, as seen in Figure 7(a) and 7(d). Therefore only one cache line needs to be updated on average during the deletion process of RH-backshift. Hopscotch hashing and Linear Probing need to mark the bucket as deleted in some way, requiring one cache line update as well. From this observation it can be concluded that even if RH-backshift has an apparent disadvantage during deletion compared to other algorithms, in the end it has a similar I/O usage than the other algorithms.

![aligned dsb_mean](https://i0.wp.com/codecapsule.com/wp-content/uploads/2014/05/aligned-dsb_mean.png?resize=704%2C575)

Figure 7: 95th percentile of the aligned distance to shift bucket (DSB) for Linear Probing, Hopscotch Hashing, and Robin Hood hashing, over four test cases

## 5. Conclusion

I choose not to use Hopscotch hashing, because it has lower performance compared to Robin Hood hashing, and more importantly because the neighborhood clustering effect makes Hopscotch hashing unreliable in practice. Indeed, there is always a chance for the distribution of the hashed keys to create a cluster in one of the neighbourhoods at any load factor, making bucket swaps impossible unless the table is resized [[3]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref). Using other neighborhood representations such as linked-list or shadow will minimize the chances of neighbourhood clusters, but won’t completely get rid of the problem. Moreover, the linked-list and shadow representations both force the data to be stored in non-contiguous memory locations, losing cache locality which is one of the major advantages of open-addressing hash tables.

Based on the analysis of the data presented above, I have chosen to use Robin Hood hashing to be the algorithm for my key-value store project. I have also recently done some research on solid-state drives [[9]](https://codecapsule.com/2014/05/07/implementing-a-key-value-store-part-6-open-addressing-hash-tables/#ref), and with that knowledge, I am now ready to finally implement that key-value store, which I will be doing over the next few months. Once done, I will write another article to describe who the hash table is stored in a way that is optimized for solid-state drives.

## Join my email list

## 6. References

[1] [Cuckoo Hashing, Emmanuel Goossaert](http://codecapsule.com/2013/07/20/cuckoo-hashing/)  
[2] [Cuckoo hashing, Pagh and Rodler, 2001](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.25.4189)  
[3] [Hopscotch Hashing, Emmanuel Goossaert](http://codecapsule.com/2013/08/11/hopscotch-hashing/)  
[4] [Hopscotch Hashing, Herlihy et at., 2008](http://mcg.cs.tau.ac.il/papers/disc2008-hopscotch.pdf)  
[5] [Source code from the authors of the original hopscotch hashing paper](https://sites.google.com/site/cconcurrencypackage/hopscotch-hashing/download-1)  
[6] [Robin Hood Hashing, Emmanuel Goossaert](http://codecapsule.com/2013/11/11/robin-hood-hashing/)  
[7] [Robin Hood Hashing: backward shift deletion, Emmanuel Goossaert](http://codecapsule.com/2013/11/17/robin-hood-hashing-backward-shift-deletion/)  
[8] [Robin Hood Hashing, Pedro Celis, 1986](https://cs.uwaterloo.ca/research/tr/1986/CS-86-14.pdf)  
[9] [Coding for SSDs](http://codecapsule.com/2014/02/12/coding-for-ssds-part-1-introduction-and-table-of-contents/)  
[10] [https://github.com/goossaert/hashmap](https://github.com/goossaert/hashmap)  
[11] [http://codecapsule.com/hashmap-data/](http://codecapsule.com/hashmap-data/)  
[12] [https://github.com/spotify/sparkey](https://github.com/spotify/sparkey)

Published in[Algorithms and Programming](https://codecapsule.com/category/algorithms/ "View all posts in Algorithms and Programming")[Implementing a Key-Value Store](https://codecapsule.com/category/implementing-a-key-value-store/ "View all posts in Implementing a Key-Value Store")