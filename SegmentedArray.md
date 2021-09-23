# Binary Sorted Arrays

## Preface

Thought of this algorithm during an interview. Obviously too complicated to implement. Also doesn't work as good as I hoped.

## TL; DR

I'm proposing a range-query data structure with the following properties, in comparison,

|               |Insert             | Range Query                      |  Query      | Delete      |
|---------------|-------------------|----------------------------------|-------------|-------------|
|Segmented Array|amortized O(log n) |O((log n)^2) + output_size        |O((log n)^2) |O((log n)^2) |
|Balanced Tree  |O(log n)           |O(log n) + output_size            |O(log n)     |O(log n)     |
|Skip List      |Avg. O(log n)      |Avg. O(log n) + output_size       |Avg. O(log n)|Avg. O(log n)|

Although big-O analysis is worst, this data structure is **easy to store on disk, and can scale past memory limit**. It also only has a set of sorted arrays, so less branching and pointer dereferencing during traversal. 

## Main body

The Segmented Array is a set of **sorted** arrays. Each array has a length power of 2. Of each possible length, there is only 1 array. For example we can have a set of lengths {1, 2, 4, 8}, {1, 4, 8}, but not {1, 1, 2}.

We can maintain this DS easily by merge sort. When we have 2 arrays of the same length, they merge into an array of the next order. Range query is binary search each array in the set and combine the results.

## Analysis
### Insert(x)
1. Create a new array, containing the new element.
2. Insert the new array into the set of sorted arrays.
3. While exists 2 arrays of same length, merge them into an array of the next order.

Steps 1, 2 take O(1) time.

Step 3 needs amortized analysis: Every 2^n th insert needs to perform a merge to the 2^n th order array. Averaging to (2^-n) * (2^n) = 1 per insert. There are log(n) arrays in our dataset, so step 3 is O(log n).

Total is O(log n).

### Range Query(start, end)
1. For each array,
2. Binary search for *start*
3. Keep outputting values until *end* is reached.
4. Combine results from each array.

Step 2, each binary search takes O(log n) time, with a total of O(log n) arrays. Total O((log n)^2) time.

Step 3, the total iterations is the output size.

Total above is  O((log n)^2 + output_size) time.

### Query(x)
1. Binary search each array. O((log n)^2) time.

This can be improved to O(log n) if we also maintain a set, but we won't support delete(x) later. Insert complexity is not impacted.

### Delete(x)
1. Binary search for value. O((log n)^2) time.
2. Mark element as deleted.

Here we need to loosen the array length constraint. Instead of eash array containing 2^n elements, it now contains (2^(n-1), 2^n] elements.

When a delete the 2^(n-1) th element, we can do one of the following:
1. If an array of (2^(n-2), 2^(n-1)] already exists, merge with it.
2. If not, remove deleted elements, make it the new (2^(n-2), 2^(n-1)] size array.

Merging/deleting both take 2^n/2 operations, but since we also did 2^n/2 deletes, each delete is amortalized O(1).

The bottle neck is O((log n)^2) binary search.
