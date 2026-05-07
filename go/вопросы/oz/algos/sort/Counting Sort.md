https://billjh.github.io/blog/2017/counting-sort/

A stable and linear-time sorting algorithm for items with keys within a limited size of collection. [Time: O(n+k), Space: O(n+k)]

### [](https://billjh.github.io/blog/2017/counting-sort/#Idea "Idea")Idea

The idea of counting sort is to count the numbers of items according to each distinct key, then decide each item’s position using prefix sum. The time complexity for counting sort is O(n+k) where n is the number of items and k is the number of possible keys for each item.

### [](https://billjh.github.io/blog/2017/counting-sort/#Implementation "Implementation")Implementation

Below is an implementation of counting sort in golang.

counting_sort.go[source](https://github.com/billjh/algo/blob/master/sorting/couting_sort.go)

|   |
|---|
|package sorting  <br>  <br>// CountingSort Time: O(n+k), Space: O(n+k)  <br>func CountingSort(arr []int, k int) {  <br>	c := make([]int, k)  <br>	for i := 0; i < len(arr); i++ {  <br>		c[arr[i]]++  <br>	}  <br>	// pre-fix sum  <br>	for i, sum := 0, 0; i < k; i++ {  <br>		// tmp := c[i]  <br>		// c[i] = sum  <br>		// sum += tmp  <br>		sum, c[i] = sum+c[i], sum  <br>	}  <br>	sorted := make([]int, len(arr))  <br>	for _, n := range arr {  <br>		sorted[c[n]] = n  <br>		c[n]++  <br>	}  <br>	copy(arr, sorted)  <br>}|

_References:_

1. [https://en.wikipedia.org/wiki/Counting_sort](https://en.wikipedia.org/wiki/Counting_sort)
2. [https://github.com/billjh/algo/tree/master/sorting](https://github.com/billjh/algo/tree/master/sorting)