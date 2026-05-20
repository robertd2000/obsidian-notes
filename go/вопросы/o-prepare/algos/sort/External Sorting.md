## An Algorithm In External Sorting

> In this article we discuss about two main types of sorting based on the volume of data to be sorted versus the capacity of primary memory : _Internal, External Sorting_. Specifically we discuss in detail about **_External Sort merge_** which is an external sorting algorithm, an extension of the same and perform **_cost analysis_** on it.

## **Overview** [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)

①. [Sorting](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#1022)  
②. [Internal Sorting](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#0b1b)  
③. [External Sorting](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#67c3)  
④. [External Sort-Merge](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#614c)  
⑤. [Extension of External Sort-Merge](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#01bf)  
⑥. [Example](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#5f18)  
⑦. [Cost Analysis](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#3598)  
⑧. [Summary](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#529a)  
⑨. [References](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#14f8)

### Sorting [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#1022)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
**Sorting** is a task of organizing data in a particular order(_ascending or descending_). Sorting is a classic, fundamental yet crucial process in solving many problems. It is used as an intermediate step in some conventional tasks such as _duplicate filtering_, _searching_.

![](https://miro.medium.com/v2/resize:fit:750/1*x9mcFZhn0dYKAkZe0QWdGQ.gif)

Sorting

In databases it is used as an intermediate in _grouping_ with aggregation, _sort-merge_ join, union and intersect set operations. It is also the first step in _bulk loading_ of [B+ tree](https://medium.com/@bnsk1998/ba46e7b1f65f) index.

Based on the volume of data, sorting can be classified into two types:

①. Internal Sorting

②. External Sorting

### Internal Sorting [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#0b1b)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
**Internal Sorting** **:** Sorting data by first fetching all the data items from secondary memory (such as hard disk, or a tape storage) to primary memory and then using a sorting algorithm (such as insertion sort, quick sort, etc…) to sort them is called _internal sorting_.  
**_For example_** : Sorting 1GB or 2GB data in a 4GB RAM machine. This type of sorting is feasible only when the data volume to be sorted is less than or equal to the capacity of primary memory available.

> **What if** the data volume to be sorted is more than the capacity of primary memory?
> 
> External Sorting is the solution.

![](https://miro.medium.com/v2/resize:fit:750/1*HRngLo3CQM7-w5nyWGVQmw.gif)

**External Sorting** is the solution when we have to sort large amounts of data. Data that doesn’t fit into primary memory of the machine on which sorting is done.

### External Sorting [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#67c3)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
**External Sorting :** Sorting data even when the volume of data is enormous when compared to storage available in the primary memory.  
For example : Sorting 8GB of data or more in a 4GB RAM machine.  
There are various algorithms for this type of sorting, such as _distribution sorting_, _external sort-merge_, _replacement selection_, etc… **_External sort-merge algorithm_**, which is the most commonly used algorithm for external sorting is discussed in this article.

### External Sort-Merge Algorithm [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#614c)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
**_External sort-merge_** algorithm consists of 2 stages :  
In the **_first stage_**, all of the data volume is divided into groups of blocks that fit in primary memory and internal sort(quick sort, insertion sort) is performed on each of these groups of blocks.  
The process of performing internal sort on these groups of blocks that fit in primary memory is called a **_run_**. The result of the internal sort is written back into the secondary memory(disk or tape).  
In **_second stage_** a block from each run is merged into an output block until all the input blocks from all the runs are merged to obtain the final result.

Let us see what happens in each stage in detail.  
Let **_P be the number of blocks_** in the primary memory that is available for sorting task. This will be the number of disk blocks that can be used to perform internal sorting.

S**tage 1** : A number of **_sorted runs are created_**; each run is sorted, but contains only some of the records of the relation.

**Pseudo code :**i = 0;  
**repeat**  
read smaller of the relations among P blocks of the relation, or the rest of the relation;  
sort the in-memory part of the relation;  
write the sorted data to run file Ri ;  
i = i + 1;  
until the end of the relation

S**tage 2** : The runs from stage 1 are **_merged_** in this stage. Suppose, for now, that the total number of runs, N < P, so that we can allocate one block to each run and have space left to hold one block of output.

**Pseudo code :**read one block of each of the N files Rᵢ into a buffer block in memory;  
**repeat**  
choose the first tuple (in sort order) among all buffer blocks;  
write the tuple to the output, and delete it from the buffer block;  
if the buffer block of any run Rᵢ is empty and not end-of-file(Rᵢ)  
   then read the next block of Rᵢ into the buffer block;  
until all input buffer blocks are empty

The output of the _stage_ 2 is the _sorted output of the input data_. The output could be written back into the disk or just buffered. The latter reduces disk write operations, therefore we consider from now on that output is buffered.  
The merge operation in stage 2 is a generalization of the two-way merge used by the standard in-memory merge sort algorithm. This merge operation in stage 2 merges N runs, and therefore called _N-way merge_.

Below figure gives an overview of both the stages and the overall flow of data through the stages.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*74EE8dR7p9f-OeIv8Z_j3A.jpeg)

**Overview of the 2 stages in External sort-merge** and the overall flow of data through the stages.

### Extension of External Sort-Merge [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#01bf)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
Until now we assumed that N < P, _what if_ there P or more runs are generated in the first stage i.e, N > P. Then it is **_not possible_** to allocate a block for each run during the _merge stage_. Therefore, the merge operation must proceed in multiple passes. Since there is enough memory for P-1 input buffer blocks, each merge can take P-1 runs as input.

## Get Veerendra Raj Kumar’s stories in your inbox

Join Medium for free to get updates from this writer.

Subscribe

Remember me for faster sign in

In the initial pass the first P-1 runs are merged to get a single run for the next pass. Then, the next P-1 runs are merged similarly, and so on, until all the initial runs are processed.

At this point, the number of runs is reduced by a factor of N’ = N-(P-1). If N’ the reduced number of runs is still greater than or equal to P, another pass is made, with the runs created by the first pass as input.  
Each pass reduces the number of runs by a factor of P − 1 from the previous pass. The passes are repeated until the number of runs is less than P. Then a final pass is done to generate sorted output.

Below figure gives an overview of both the stages over various number of merge passes and the overall flow of data through the stages.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*kpfT907CbNOzC4lQsMK5nQ.jpeg)

Overview of the 2 stages and a number of merge phases in **Extended External sort-merge**.

### Example [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#5f18)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
Lets see an example of the above learned concept :  
Given input data(relation), the task is to sort the given data. Assume that _only one tuple fits in a block_, and that primary memory can hold _at most three blocks_, which implies that in _Stage 2_ , **_two blocks are used for input and one for output._**

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*bdtsNydS01Gft80xTPb1iA.jpeg)

**Example** of Extended External Sort-Merge

### **Cost Analysis** [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#3598)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
Let us first compute _disk-access_ cost for the external sort–merge algorithm discussed until now:  
Let bᵣ denote the number of blocks containing records of relation r.  
In Stage 1 every block of the relation is read and written into the disk, giving a total of 2×bᵣ block transfers. The initial number of runs is ⌈bᵣ / P⌉.  
Since the number of runs decreases by a factor of P-1 in each merge pass, the total number of merge passes required is ⌈log ₚ ₋₁( bᵣ / P )⌉.

In general each of these passes reads every block of the relation once and writes it out once. Therefore, the total number of block transfers for external sorting of the relation. Is given by the following relation :

**bᵣ ( 2×⌈logₚ₋₁( bᵣ /P )⌉+ 1 )**

Applying this equation to the above example, we have br = 12, P = 3, which yields a total of 12 ×(⌈log₂( 12/ 3)⌉ + 1) = 12 × (4 + 1) = _60 block_ transfers.

Overall cost is given majorly by **_disk access_** time, but **_disk seek_** time also plays a significant role in the overall cost. Now that we have obtained an expression to compute disk access costs, let us concentrate on the disk-seek costs.

In Stage 1, disk seeks takes place _once for reading data_ for each of the runs as well as for _writing the runs_. During the merge phase, suppose bʙ blocks are read at a time from each run, then each merge pass would require around bᵣ/bʙ seeks for reading data.

Although the output is written sequentially, if it is on the same disk as the input runs, the head may have moved away between writes of consecutive blocks. Thus we have to add a total of 2 × ⌈bᵣ / bʙ⌉ seeks for each merge pass except the final pass (since we assume the final result is buffered, and not written back to disk). Therefore the total number of seeks is then given by:

**2×⌈bᵣ/P⌉ + ⌈bᵣ/bʙ⌉ × (2×⌈logₚ₋₁(bᵣ /P)⌉ - 1)**

Applying this equation to the example discussed above, where bᵣ = 12 , P = 3, bʙ = 1, we get total number of disk seeks as :  
2×⌈12 /3⌉+ ⌈12 /1⌉ × (2× ⌈log2(12/3)⌉-1 )  
= 8 + 12 × (2 × 2 -1 ) = 44 disk seeks.

### Summary [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#529a)

[[Overview](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#be14)]  
We first saw **sorting** being used both as an _independent and intermediate step_ in many of the processes or tasks involved in various database concepts, which made it crucial and yet fundamental problem to be discussed. Based on the _volume of data_ to be sorted and the _capacity of primary memory_(RAM) we divided sorting into two types, namely : **Internal** and **External Sorting**.  
We then focused on **External Sort-Merge** widely used algorithm in external sorting. Went through the _2 stages_ of **creation of runs, merging the runs**, to obtain the final result.  
We also went through an extended version of External sort-merge algorithm which consisted of many passes in the merge phase. Then **Cost analysis** was done on the discussed algorithm.

### References [☍](https://veerendra-raj-kumar.medium.com/external-sort-merge-an-algorithm-in-external-sorting-e120a245e735#14f8)

- Database System Concepts by Abraham Silberschatz, Henry F. Korth, , S. Sudarshan; 6th Edition McGraw Hill Publications.
- [https://doi.org/10.1007/s10586-018-2860-1](https://doi.org/10.1007/s10586-018-2860-1)
- [http://www.csbio.unc.edu/mcmillan/Media/Comp521F10Lecture17.pdf](http://www.csbio.unc.edu/mcmillan/Media/Comp521F10Lecture17.pdf)
- [https://opendsa-server.cs.vt.edu/ODSA/Books/Everything/html/ExternalSort.html](https://opendsa-server.cs.vt.edu/ODSA/Books/Everything/html/ExternalSort.html)
- [http://www.gvpcew.ac.in/LN-CSE-IT-22-32/CSE-IT/2-Year/22-ADS/ADS-AUK-UNIT-1.pdf](http://www.gvpcew.ac.in/LN-CSE-IT-22-32/CSE-IT/2-Year/22-ADS/ADS-AUK-UNIT-1.pdf)
- [https://en.wikipedia.org/wiki/External_sorting](https://en.wikipedia.org/wiki/External_sorting)

### Much more on Database System Concepts from others

- [https://medium.com/@bnsk1998/bplus-tree-ba46e7b1f65f](https://medium.com/@bnsk1998/bplus-tree-ba46e7b1f65f)
- [https://youtu.be/S_ligfWrgO4](https://youtu.be/S_ligfWrgO4)
- [https://medium.com/@ssssubhank/deadlock-handling-beee7143cb8a](https://medium.com/@ssssubhank/deadlock-handling-beee7143cb8a)
- [https://medium.com/@saisuraj2310/two-phase-locking-b0c4c22856a3](https://medium.com/@saisuraj2310/two-phase-locking-b0c4c22856a3)

[

External Sorting

](https://medium.com/tag/external-sorting?source=post_page-----e120a245e735---------------------------------------)

[

External Sort Merge

](https://medium.com/tag/external-sort-merge?source=post_page-----e120a245e735---------------------------------------)

[

Sorting In Databases

](https://medium.com/tag/sorting-in-databases?source=post_page-----e120a245e735---------------------------------------)

[

Internal Sorting

](https://medium.com/tag/internal-sorting?source=post_page-----e120a245e735---------------------------------------)