Heap-sort is a beautiful sorting algorithm. It uses a max-heap to sort a sequence of numbers or other elements with a defined order relation. In this article we’ll deep-dive into the **Go standard library** heap-sort implementation.

# Max-heap

First a short recap on [**binary max-heaps**](https://en.wikipedia.org/wiki/Heap_\(data_structure\)). A max-heap is a container that provides its maximum element in O(1) time, adds an element in O(log n), and removes the **maximum element** in O(log n).

Max-heaps are **almost-full** binary trees, where **every node is greater or equal to its children**. I’ll refer to the latter as **the heap property** throughout the article.

Together, these two properties define a max-heap:

![Image](https://www.freecodecamp.org/news/content/images/2019/08/image-117.png)_A Max heap. By Ermishin — Own work, CC BY-SA 3.0, [https://commons.wikimedia.org/w/index.php?curid=12251273](https://commons.wikimedia.org/w/index.php?curid=12251273)_

In heap algorithms, a max-heap is represented as an array. In the array representation, the children of the node in index `i` are located in indices `2*i+1` and `2*i+2`. This chart from Wikipedia shows the array representation:

![Image](https://www.freecodecamp.org/news/content/images/2019/08/image-118.png)_Max-heap represented as an array. By Maxiantor — Own work, CC BY-SA 4.0, [https://commons.wikimedia.org/w/index.php?curid=55590553](https://commons.wikimedia.org/w/index.php?curid=55590553)_

## Building a heap

An array can be converted into a max-heap in O(n) time. Amazing, isn’t it? This is the algorithm:

1. Treat the input array as a heap. It doesn’t satisfy the heap property yet.
2. Iterate the nodes of the heap starting from the second-to-last level of the heap — that’s one level above the leaves — going back to the root.
3. For each node encountered, propagate it down in the heap, until it is greater than both its children. When propagating down, always swap with the greater child.

That’s it! You’re done!

Why does it work? I’ll try to convince you with this hand-wavy proof (Feel free to skip though):

- Take a tree node `x`. Because we iterate the heap from back to front, when we reach it, the subtrees rooted in both its children already satisfy the heap property.
- If `x` is greater than both its children, we’re done.
- Otherwise, we swap `x` with its biggest child. That makes the new root of the subtree greater than both its children.
- If `x` doesn’t satisfy the heap-property in its new subtree, the process will be repeated until it does or becomes a leaf, in which case it won’t have any children.

This is true for all nodes in the heap, including the root.

# Heap-sort algorithm

Now for the main course — heap-sort.

Heap-sort works in 2 steps:

1. Builds a max-heap from the input array using the algorithm I showed above. This takes O(n) time
2. Pops elements from the heap into the output array, filling it from the back to the front. Every removal of the maximum element from the heap takes O(log n) time, which adds up to O(n * log n) for the entire container.

A cool property of the Go implementation, is that it uses the input array to store the output, thus saving the need to allocate O(n) memory for the output.

# Heap-sort implementation

The Go sort library supports any collection that is **indexed by integers**, has a **defined order relation** on its elements, and **supports swapping** elements between two indices:

```go
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
```

Naturally, any contiguous container of numbers can satisfy this interface.

Now let’s take a look at the body of `heapSort()`:

```go
func heapSort(data Interface, a, b int) {
    first := a
    lo := 0
    hi := b - a

    // Build heap with greatest element at top.
    for i := (hi - 1) / 2; i >= 0; i-- {
        siftDown(data, i, hi, first)
    }

    // Pop elements, largest first, into end of data.
    for i := hi - 1; i >= 0; i-- {
        data.Swap(first, first+i)
        siftDown(data, lo, i, first)
    }
}
```

The function’s signature is a bit cryptic, but looking at the first 3 lines clear things up:

- `a` and `b` are indices into `data`. `heapSort(data, a, b)` sorts `data` in the half-open range `[a, b)`.
- `first` is a copy of `a`.
- `lo` and `high` are indices normalized by `a` — `lo` always starts at zero, and `hi` at the size of the input.

Next, the code builds the max-heap:

```go
// Build heap with greatest element at top.
for i := (hi - 1) / 2; i >= 0; i-- {
  siftDown(data, i, hi, first)
}
```

As we saw earlier, the code scans the heap from one level above the leaves and uses `siftDown()` to propagate the current element down until it satisfies the heap property. I’ll go into `siftDown()` in more detail below.

At this stage, `data` is a max-heap.

Next, we pop all the elements to create the sorted array:

```go
// Pop elements, largest first, into end of data.
for i := hi - 1; i >= 0; i-- {
  data.Swap(first, first+i)
  siftDown(data, lo, i, first)
}
```

In this loop, `i` is the last index in the heap. On each iteration:

- The heap’s maximum `first` is swapped with the last element of the heap.
- The heap property is restored by propagating the new `first` down until it satisfies the heap property.
- The heap size `i` is reduced by one.

In other words, we are filling the array from the back to the front, starting from the largest element, to the 2nd element in size all the way to the smallest element. The result is the sorted input.

## Maintaining the heap property

Along the post I mentioned using `siftDown()` to maintain the heap property. Let’s see how it works:

```go
// siftDown implements the heap property on data[lo, hi).
// first is an offset into the array where the root of the heap lies.
func siftDown(data Interface, lo, hi, first int) {
    root := lo
    for {
        child := 2*root + 1
        if child >= hi {
            break
        }
        if child+1 < hi && data.Less(first+child, first+child+1) {
            child++
        }
        if !data.Less(first+root, first+child) {
            return
        }
        data.Swap(first+root, first+child)
        root = child
    }
}
```

This code propagates the element in `root` all the way down the tree until it is larger than both of its children. When going down a level, the element will be swapped with its greater child. That is to make sure that the new parent node is greater than both its children:

![Image](https://www.freecodecamp.org/news/content/images/2019/08/image-119.png)_The parent ‘3’ is swapped with the greatest child ‘10’_

The first few lines calculate the index of the first child and check that it exists:

```go
child := 2*root + 1
if child >= hi {
  break
}
```

`child >= hi` means that the current `root` is a leaf, so the algorithm stops.

Next, we pick the greater of the two children:

```go
if child+1 < hi && data.Less(first+child, first+child+1) {
  child++
}
```

Since any node’s children are next to each other in the array, `child++` selects the second child.

Next, we check if the parent is indeed smaller than the child:

```go
if !data.Less(first+root, first+child) {
  return
}
```

If the parent is greater than it biggest child, we are done so we return.

Lastly, if the parent is smaller than the child we swap the two elements and increment `root` to prepare for the next iteration:

```go
data.Swap(first+root, first+child)
root = child
```

# Conclusion

This is the third article where I read an unfamiliar piece of code and try explain it. I like this kind of experience because it teaches me how to read code and how to communicate about it. Please leave your comments and feedback below!