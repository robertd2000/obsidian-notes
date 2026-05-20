https://medium.com/@akindipejohn2017/implementing-radix-sort-in-golang-cce5cb31b852

The fastest comparison sorts (such as quicksort and merge sort) have a time complexity of **O(n log n)** and mathematical proofs exist to show that these sorts cannot get any faster than this. Honestly, we may not need to be faster than that anyway, but it doesn’t hurt to consider other possible sorting algorithms which may offer gains in processing time, however marginal and albeit in only a handful of circumstances. One of such sorting algorithms is **Radix Sort** (also, the sort is rad — all pun intended — and fun to implement, what’s not to love😁)

Let’s start then, by explaining the logic behind radix sort. Given the slice of integers:

![](https://miro.medium.com/v2/resize:fit:569/1*sCoUHzgSGuUb2FGfEdRPRA.png)

Slice of integers to sort

How do we utilize the radix sort algorithm to sort this slice of integers:

- The algorithm will not directly compare the numbers in the slice (since it is not a comparison algorithm), instead it will repeatedly group the numbers in the slice inside a **bucket** (we will make this a map of integers to a slice of integers), and then rearrange them in the slice with specific ordering, until the slice becomes sorted.
- In each pass, the algorithm considers the LSD (Least Significant Digit) of each number and groups them in the bucket based on their LSD and then rearranges them in the slice.
- The number of iterations will depend on the number of digits in the largest number in the slice. The rationale behind this is explained below. In the first iteration, we consider the digits in the unit position, then in the next iteration, the digits in the tens position, and so on.
- In the first pass, the least significant digit in 20 is 0, hence, we add it to our bucket under the key 0:  
    { 0: [20] }  
    Continuing from here, the LSD in 6 is, well 6, we thus group it in the bucket under the key 6: { 0: [20], 6: [6] }.  
    The LSD in 25 is 5, we thus group it in the bucket under the key 5 { 0: [20], 6: [6], 5: [5] }.  
    Following this logic, the LSDs in 5, 210, 4 and 1 are 5, 0, 4 and 1 respectively and they are grouped appropriately inside our bucket:

![](https://miro.medium.com/v2/resize:fit:268/1*_b-Jlqbh4JuFMyXakPZuVQ.png)

Our bucket after the first pass, the elements are grouped in the bucket based on the digits in the unit position

Now that we have placed these elements inside the bucket, all we do now, is rearrange these elements back into the slice, we start from the 0 group and then unto higher groups. Eventually when we rearrange the numbers we get:

![](https://miro.medium.com/v2/resize:fit:591/1*ZsYFaHSjr7W3uI6RoKTkAA.png)

The slice at the end of the first pass

It is very important that when we re-arrange elements from the bucket into the slice, we respect the ordering in which they are rearranged into the slice. Elements in the smaller bucket groups must be considered before elements in higher bucket groups; if a bucket group contains multiple elements (such as groups 0 and 5 in the above example), the elements must be arranged in the slice such that elements from the zero index are added to the slice, before elements from higher indexes. Notice that 20 comes before 210 in the re-organized slice and 25 comes before 5. This is important because radix sort is a stable sort, that is if the slice contains two identical numbers, then they should preserve their ordering in the original slice as they do after the slice is sorted. With this we have completed the first pass and can continue on to the second.

- In the second pass, we consider the next least significant digit (that is, the number at the tens position).

As mentioned earlier, the number of passes this algorithm needs to sort any given slice is equal to the number of digits in the largest number in the slice. What this means is that in the slice above, since the largest number is 210, and it has 3 digits, we’ll need to arrange our slice of values into the bucket 3 times. After the first pass, we know that our slice now looks like this:

![](https://miro.medium.com/v2/resize:fit:591/1*ZsYFaHSjr7W3uI6RoKTkAA.png)

We will now place the slice values into buckets based on the next LSD (warning: what this means has already been defined above 😂), essentially we’ll be placing the slice elements into the bucket based on the number in the tens position. For any number without a digit in the tens position (i.e. all single-digit numbers), we will place them under the “0” key. The first element in the slice now is “20”, the number in its tens position is 2, it will be placed in the map under the key “2”, the number “210” will be placed under the key “1”, numbers 1, 4, 5 and 6 will be placed under the key “0”, (being single-digit numbers). 25 will be placed under key “2”. Eventually our map will look like this.

![](https://miro.medium.com/v2/resize:fit:282/1*DzmDnsB2fX5lpLngQLycqQ.png)

The bucket at the second pass — considering the digits in the tens position

> It is very important to realize at this point that our bucket will look different after each pass, that is, it will contain keys different for each pass, because the digits in the tens position of the numbers being sorted is not very likely going to be the same as was in the unit position (or any other position). Thus, at the end of each pass, we flush the bucket and make it empty in preparation for the next pass.

After placing them in the map, we retrieve them and put them in a slice, again we must maintain order in retrieving these elements back into the slice. when we do this, our slice now becomes:

![](https://miro.medium.com/v2/resize:fit:601/1*bIQrCYYJK6fkj6Lmpa57Gg.png)

The slice after the second pass.

Pun alert!, the eagle-eyed among may have noticed our sort is already taking flight: all the single-digit elements are sorted! Like we mentioned the number of passes (each pass corresponds to putting these elements into the bucket and retrieving them from the bucket) will equal the number of digits in the largest number among the elements, which is “3” in this case. Ahoy mateys, on to the final pass, ARE YE READY!

- In the final pass — which is the 3rd pass in this case — we are considering the numbers in the next least significant digit (the hundreds position).

Now, we will use the same logic as we have been using, that is, numbers which do not have a digit at the hundred position, will be put in the zero bucket inside the map. What this means is that single-digit and double-digit numbers will be put in the zero bucket inside the map. Thus, when we sort the current slice which currently looks like:

![](https://miro.medium.com/v2/resize:fit:601/1*bIQrCYYJK6fkj6Lmpa57Gg.png)

into the bucket based on the digit in the hundred position we will get:

![](https://miro.medium.com/v2/resize:fit:393/1*oDH-xj8GSATAHpEPI8YnPQ.png)

The bucket at the third pass — considering the digits in the hundred position.

> Notice again how different our bucket looks at the third pass, different than it looked at the first and second pass. This again re-emphasizes the need to flush the bucket at the end of every pass, because we know the bucket will not likely contain the same keys in the next pass.

And when we put these back into the slice, we get (inserts drumroll effect 😁) the slice like so:

![](https://miro.medium.com/v2/resize:fit:594/1*uQn_PcKw3fO6hWx5QaXFhw.png)

Final sorted slice after third pass. Three passes required due to number of digits — 3 — in largest number: 210

Perfectly sorted, as all things should be. Now we can rest and watch the sun rise on a grateful Arrayverse.

Having explained the logic, let us look at the code to execute it.

### **Radix Sort Code**

func radixSort(intSlice []int) []int {  
 if len(intSlice) <= 1 {  
  return intSlice  
 }  
 //get the max number in the slice  
 maxInt := getMaxInt(intSlice)  
  
 //get the num of digits in the max num  
 numOfDigits := getDigits(maxInt)  
  
 //create bucket  
 bucket := map[int][]int{}  
  
 //now we know how many times we will  
 //insert and get from the bucket  
 //i.e the numOfDigits times  
 for i := range numOfDigits {  
  insertToBucket(i, intSlice, bucket)  
  getFromBucket(intSlice, bucket)  
 }  
  
 return intSlice  
}

Let’s work through the above:

- If the slice contains 1 or no element, we immediately return it. These slices are already sorted.
- We get the maximum number in the slice.
- Get the number of digits in the maximum number
- Create a bucket
- Based on the number of digits in the maximum slice member, we iteratively insert the elements in the slice to the bucket and get from the bucket into the slice (in the same iteration).
- We then return the slice, which is now sorted.

_The below is the function to get the maximum integer in the slice. Pretty simple_

// This function returns the maximum element in an int slice  
func getMaxInt(intSlice []int) int {  
 maxInt := 0  
 for _, v := range intSlice {  
  maxInt = max(maxInt, v)  
 }  
 return maxInt  
}

_This gets the number of digits in the maximum integer: The comments explain the logic._

//This function returns the number of digits in an integer  
//It does this by taking its logarithm, adding 1 and flooring the result,   
t//be equal to the number of digits in the num  
//e.g. 10 => log(10) is 1. 1+1 = 2. flooring 2 gives us 2.  
func getDigits(num int) int {  
 if num == 0 {  
  return 1  
 }  
 //remove the negative sign (in case num is negative)  
 positiveNum := int64(math.Abs(float64(num)))  
 floatNumOfDigits := math.Floor(math.Log(float64(positiveNum)) + 1)  
 return int(floatNumOfDigits)  
}

### The Insert to Bucket Function

func insertToBucket(loopNumber int, intSlice []int, bucket map[int][]int) {  
 for _, v := range intSlice {  
  vStr := strconv.Itoa(v)              //20 => "20"  
  vStrSlice := strings.Split(vStr, "") //"20" => ["2", "0"]  
  digit := 0  
  //The digit is automatically assigned a value of 0  
  //This will cater for the conditions when a value does not  
  //have a digit in a particular position (maybe in the tens, hundred,  
  //thousand position etc.) In such situations, it is guaranteed that  
  //the loopnumber will be equal to or greater than the  
  //length of the value's slice.  
  if loopNumber >= len(vStrSlice) {  
   bucket[digit] = append(bucket[digit], v)  
   continue  
  }  
  
  digit, _ = strconv.Atoi((vStrSlice[len(vStrSlice)-(loopNumber+1)])) //loopnumber starts at 0  
  //["2", "7"] => ["7"] for unit (loopnumber => 0)  
  //["2", "7"] => ["2"] for tens (loopnumber => 1)  
  //["2", "7"] => ["0"] for hundred (loopnumber => 2)  
  
  bucket[digit] = append(bucket[digit], v)  
 }  
}

The logic for this function has already been explained, but let’s step through it again, using the same slice as we did above:

![](https://miro.medium.com/v2/resize:fit:569/1*sCoUHzgSGuUb2FGfEdRPRA.png)

Original slice to sort

- We range over the integer slice and obtain a value
- We stringify the value and turn it into a string slice
- We assign the integer variable “digit” a default value of 0. This will be the default value if a number doesn’t contain a digit in a particular position, such as for a single-digit number when we are considering digits in the tens position.
- The function accepts a loopNumber parameter which tells us in which iteration (and consequently what digit position we are considering) we are. i.e if the loopNumber is 0, that means we are considering digits in the unit position.
- If the loopNumber is greater than or equal to the length of the stringified slice of the number we’re considering, we give that number the value 0 for that position and assign it to the bucket under the key 0. For instance, if loopNumber is 2 which means we are considering the digit in the hundred position, the stringified slice of 20 will have a length of 2, (the loopNumber is equal to the length of the stringified slice, this means that the number doesn’t have a digit in the hundred position) and we assign it the default value 0.
- If the loopNumber is less than the length of the stringified slice, it means the number we are considering definitely has a digit in the position we’re considering (either unit, tens, hundreds and so on).
- We then have to get the digit at the position corresponding to the loopNumber. To do this, we do some calculation. Again using the number 20 as an example, its stringified slice is [“2”, “0”]. At loopNumber 0 (i.e. considering the unit digit), we know this digit in the number 20 is 0 and is at index 1. The formula to get it will be:

- Considering ["2", "0"]  
stringifiedSlice[len(stringifiedSlice) - (loopNumber + 1)]  
  
- In this case, this will resolve to:  
stringifiedSlice[2 - (0 + 1)] = stringifiedSlice[2 - 1] = stringifiedSlice[1]  
stringifiedSlice[1] => 0   
  
- Thus we put the number 20 under the  
bucket key 0 in our map. Rememer that we are using a map[int][]int. Thus we are  
appending 20 to that slice.  
  
- In the next iteration, loopNumber will be 1. The loopNumber is less than the  
length of the stringified slice ["2", "0"]. This means that the number 20  
has a digit in the position corresponding to loopNumber 1 (i.e. tens).  
  
- We get this digit at this position using the same formulae as above.  
stringifiedSlice[len(stringifiedSlice) - (loopNumber + 1)]  
which will resolve to  
stringifiedSlice[2 - (1 + 1)] = stringifiedSlice[2 - 2] = stringifiedSlice[0]  
stringifiedSlice[0] => 2  
  
- Having correctly identified 2 as the digit in the tens position for the   
number 20, we put it (i.e. the number 20) this time in the slice corresponding  
to the bucket number 2.  
  
- If it happens that we have a third iteration, loopNumber is 2 which means we   
are considering the digit in the hundred position, ["2", "0"] has a length of   
2, (the loopNumber is equal to the length of the stringified slice and we   
trigger the "if" check), this means that the number 20 doesn't have a digit   
in the hundred position. Thus we default its hundred digit to 0 and append 20  
to the slice in the bucket under the key 0. Future iterations, if any, will  
all trigger the if check (since loopNumber will be greater than the length of  
the stringified slice) and the same thing already described will happen.

The above summarily describes the insert to bucket function. Next up, let’s look at the **get from bucket** implementation. At this point, you may need to get a snack, I know I need one 😋

## Get John Akindipe’s stories in your inbox

Join Medium for free to get updates from this writer.

Subscribe

Remember me for faster sign in

**The Get From Bucket Function**

_The below code shows the getFromBucket function_

func getFromBucket(intSlicePtr []int, bucket map[int][]int) {  
 //This variable will keep track of the index the loop stopped  
 //indexing at.  
 currentIdx := 0  
 //i will range from 0-9  
 for i := range 10 {  
  //if we don't have a key in the map for a specific number;  
  //continue to the next  
  if _, exists := bucket[i]; !exists {  
   continue  
  }  
  
  //pick a basket from the bucket  
  intSlice := bucket[i]  
  //pick the elements from the basket into the slice  
  //keep track of the index we stopped at  
  for _, intVal := range intSlice {  
   intSlicePtr[currentIdx] = intVal  
   currentIdx++  
  }  
  //we need to empty the bucket after each pass from the bucket  
  delete(bucket, i)  
 }  
}

We have already explained the logic for how this piece of code works, but we will step through it again:

- This function accepts a slice of integers and a bucket (which is a map of integers to a slice of integers).

> Note that slices are actually implemented as pointers to a backing array in Go. Therefore, while it is true that the slice we are actually passing to this function is a copy of the slice we are sorting. This copy is actually a pointer to the same backing array. Hence, changes we make to this copy will be made in the same backing array.

- This function is called after we have inserted the slice elements into buckets according to digits in (either the unit, tens, hundred position and so on). We call this function to get the elements from the bucket and back into the slice.
- Now it is very important that when we get the elements back from the bucket and into the slice, we ensure the following: For elements that share the same bucket, the leftmost elements must be added to the slice first. Also, the bucket keys must be considered in increasing order, from 0 to 9. That is, elements under the bucket key 0 must be put back into the slice before elements under the bucket key 1.

- Firstly, we initiate a variable to hold the value of the index  
in the slice where we are currently inserting to. It starts at   
index 0.  
- We range from 0 to 9 (to correspond to the possible values of the  
bucket keys). Note that we cannot simply range over the bucket keys.  
In Go, there is no guarantee over how a range in a map goes. Every  
range will order the keys in random fashion, which will defeat the  
purpose of what we are trying to achieve. You can read about that  
here https://go.dev/blog/maps#iteration-order  
- For each value we are ranging for, we are not guaranteed that this  
value will exist in our bucket. That is, we cannot be certain that  
the bucket has a key "8" for example, if we did not find any numbers  
containing the digit "8" in their unit position. Thus we  
check if the key exists in the bucket, if it doesn't we simply  
continue to the next iteration of the loop.  
- If the key exists in the bucket, we get the intSlice associated  
with the key (remember that our bucket is a map[int][]int - meaning  
that every key is associated with a slice of ints).  
- We range over the intSlice and put its elements into the slice  
being sorted at the appropriate index (which we know because of the  
currentIdx variable which helps us keep track of which index in the  
slice is being updated). For every update, we increment the value of  
the currentIdx variable.  
- Finally, we delete the associated key (and its values) from the  
bucket. We need to do this because we need to flush the bucket  
before the next iteration of inserting to the bucket. The bucket will not  
contain the same key during each iteration, thus it must be flushed  
at the end of one iteration before the next.

> We can’t iterate over the map depending on the key ordering because of the non-deterministic behaviour of ranging over maps in Go. Read more here [https://go.dev/blog/maps#iteration-order](https://go.dev/blog/maps#iteration-order)

### Time Complexity

At this point, let us discuss the time complexity for this beautiful algorithm. Firstly, let’s consider the operations that run in each iteration of this algorithm. We know, that for each integer, we will put it into the bucket and then re-arrange them from the bucket into the slice. This gives us two operations per input, thus we have, at this point, a time complexity of **_O(2 * n)_**

![](https://miro.medium.com/v2/resize:fit:496/1*Jv_Nx63zVmIi5SW5ObEEKA.png)

We perform two operations for every input per iteration.

Continuing from there, we have to then determine, how many times the iteration will run. We know again that the number of times the iteration will occur is equivalent to the number of digits in the highest value in the input — let’s call this value: **_K_**

We can then logically deduce that the time complexity of this algorithm is: **_O(2 * n * K)._** Discarding constants, we arrive at the final representation of the time complexity:

![](https://miro.medium.com/v2/resize:fit:184/1*_i9wzULyirWpB68pTC4q1w.png)

Where “K” is the number of digits in the largest input

The time complexity remains the same in the average, best and worst case scenario.

There are a few things to consider however, with this time complexity in comparison to other sorting algorithms. Quicksort for example has a time complexity of **O(n log n).** If certain conditions are met, radix sort could perform faster than quicksort. If we consider time complexities for both algorithms, we see that they both have **_“n”_** in common, ergo, any difference in performance will be achieved as a result of the varying co-efficients. We can surmise therefore that for radix sort to be faster than quicksort, **_“K”_** (the number of digits in the largest input), must be smaller than **log n** for the same set of data. To illustrate this point, let’s do some quick maths (considering, for simplicity sake, all other factors to be equal and ignoring system variabilities), say we have an input of 256 numbers with the largest input having 3 digits, we can correctly assume the following:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:963/1*wGhb6fSR4FzwgzusQuzVoQ.png)

![](https://miro.medium.com/v2/resize:fit:738/1*K8b83xH5uksCK_y-MysWwg.png)

The number of digits in the largest input is “3”

We see from the above that in the scenario described, radix sort does almost 3 times less number of operations than quicksort. This, as a result of the fact that **“K”** (3) is less than **log n** (8). Hold the celebrations though, turns out, radix sort could also be much slower. Let’s see how.

Let’s compare radix sort and a sort with quadratic time complexity such as bubble sort which has a time complexity of **O(n²)**. There are scenarios where radix sort could result in time complexities of quadratic time. Very simply; if it happens that **_“K = n”,_** the time complexity of radix sort **O(n * k)** could then be expressed as: **O(n * n)** to give a time complexity of **O(n²).** In this scenario, radix sort will be just as slow as bubble sort or any other such sorting algorithm with quadratic time complexity. What’s more, it could even be considerably worse than quadratic time complexity if **“K > n”**, real bummer innit 😪

From the above, we can conclude that radix sort really shines in such cases when **_“K”_** is much lesser than **_“n”,_** with reducing performance as **“K”** approaches **“n”**. It goes without saying that it behooves the programmer, given particular variables, constraints and according to the dictates of the circumstance, to reach for the algorithm that solves the problem with the most efficiency.

### Summary

We have explored implementing radix sort in golang in this article. We saw that it is different from the comparison sorts and doesn’t compare individual elements to each other. Instead it achieves its sorting by doing the following:

- Repeatedly groups the elements into buckets based on digits in the unit, tens, hundreds position and so on.
- Then it regroups them into the slice, making sure to maintain order.
- The number of times it iteratively does this is equal to the number of digits in the largest number in the slice it is sorting.
- At the end of the last iteration, the slice is sorted.
- The time complexity of radix sort algorithm is **O(n * k)**, where “**K”** is the number of digits in the largest input
- Radix sort can be faster than quick sort if **“K”** is less than **“log n”**
- Radix sort could also be as slow as quadratic time algorithms if “**_K” equals “n”_**
- We must choose the appropriate algorithm for a problem after careful consideration of the peculiarities and constraints of the problem we’re trying to solve.

N.B: No birds were harmed in the making of this article. Also, any and all references to controlled substances are purely co-incidental.

N.B.B: If you can’t already tell, this article contains a number of MCU references.

The keen eyed may have observed that this algorithm doesn’t cater for negative numbers. Implementation for this is slightly different and is discussed in detail in this article: [Sorting Negatives with Radix Sort](https://medium.com/@akindipejohn2017/implementing-radix-sort-in-golang-part-2-sorting-negatives-2675a85e993c). See y’all again soon, till then, stay rad 😎