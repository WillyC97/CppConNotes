## STL Algorithms

* Speaker: Jonathan Boccara
* Talk: [105 STL Algorithms in Less Than an Hour](https://www.youtube.com/watch?v=2olsGf6JIkU&ab_channel=CppCon)

## HEAPS

In C++ a heap is a tree-like structure where the child node of every parent node is less than the value of parent.

* `make_heap` turns a range of elements into a heap.
* `push_heap` adds a value to its correct position in the heap.

A heap can be used to quickly get the max element in a collection. The max element of a heap is the first element

`pop_heap(begin(numbers), end(numbers));`

## SORTING

* `std::sort` sorts a collection
* `partial_sort`
* `nth_element`
* `sort_heap`
* `inplace_merge`

## PARTITIONING

* `parition`
* `parition_point` the point in which the collection has been split

## OTHER PERMUTATIONS

* `rotate` takes elements from the end and places them at the beginning
* `shuffle` rearranges a collection in a random order
* `reverse`

## NUMERICAL ALGORITHMS

* `count` the number of occurrences of an elements in a collection
* `accumulate` sums the contents of an array
* `partial_sum`
* `inner_product`
* `adjacent_difference` the difference between neighbours
* `sample` produces n elements of the colleciton selected randomly

* `all_of` all elements satisfy the predicate
* `any_of` at least one element satisfies the predicate
* `none_of` no elements satisfy the predicate

### Comparing two ranges
* `equal`
* `lexicographical_compare`
* `mismatch` returns a `std::pair<iterator, iterator>` at the point in which the two ranges start to differ

## SEARCHING FOR A VALUE

* `find` finds a value
* `adjacent_find` finds the first position where two of the target value appear next to each other
* `search` finds a subrange within a range
* `max_element`
* `min_element`
* `minmax_element` returns `std::pair<iterator, iterator>`
