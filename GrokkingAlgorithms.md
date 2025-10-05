## Chapter 1

### Binary Search

Binary search halves the guesses you make each iteration.

You will need at worst log2n guesses

```c++

int binarySearch(const std::vector<int>& v)
{
  const int low {};
  const int high { v.size() - 1 };

  while (low <= high)
  {
    const auto mid = (low + high) / 2;
    const auto guess = v[mid];

    if (guess == item) return mid;
    if (guess > item) high = mid - 1;
    else low = mid + 1;
  }

  return -1;
}

```

### Big O notation

Big O always measures the WORST case scenario of an algorithm.

example Big O run times:
* `O(log n)`: also known as log time. Example: Binary search.
* `O(n)`: also known as linear time. Example: Simple search.
* `O(n * log n)`: Example: A fast sorting algorithm, like quicksort
* `O(n2)`: Example: A slow sorting algorithm, like selection sort
* `O(n!)`: Example: A really slow algorithm, like the traveling salesperson

## Chapter 2

* Arrays:
  * Store memory contiguously
  * Need to reallocate memory if you need more items than the size of the array
  * **Random Access**


* LinkedList:
  * Stores the item and the address of the next item
  * Can be spread across memory
  * Insertion and deletion: Change what nodes are pointing to
  * **Sequential Access**



  |           | Arrays | Lists |
  |-----------|--------|-------|
  | Reading   | O(1)   | O(n)  |
  | Insertion | O(n)   | O(1)  |
  | Deletion  | O(n)   | O(1)  |

## Chapter 3

### The stack

* Has two operations: `push` and `pop`
* `push` **adds** to the **top** of the stack
* `pop` **takes** away from the **top** of the stack

Some shit about recursion that I couldn't be bothered to read

## Chapter 4

### Divide and Conquer

D&C uses recursion = can suck a dick

### Quicksort

```c++

std::vector<int> quicksort(const std::vector<int>& arr)
{
    if (arr.size() <= 1) return arr; // base case

    int pivot = arr[0];

    // Partition using ranges
    auto less = arr | std::ranges::views::filter([pivot](int x){ return x < pivot; });
    auto greater_equal = arr | std::ranges::views::filter([pivot](int x){ return x >= pivot; });

    // Recursively sort partitions
    std::vector<int> sorted;

    auto sorted_less = quicksort({less.begin(), less.end()});
    auto sorted_greater_equal = quicksort({greater_equal.begin(), greater_equal.end()});

    // Concatenate results
    sorted.insert(sorted.end(), sorted_less.begin(), sorted_less.end());
    sorted.insert(sorted.end(), sorted_greater_equal.begin(), sorted_greater_equal.end());

    return sorted;
}

```

### Hash Tables
