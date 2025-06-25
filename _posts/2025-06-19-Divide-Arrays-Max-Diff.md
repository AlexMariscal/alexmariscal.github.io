---
title: Divide Arrays Into Arrays by Max Difference
date: 2025-06-25 12:13:00 -0400
categories: [leetcode, programing]
tags: [c++, sorting, quicksort]
---

## The Leetcode Problem
This was a medium level problem.
You are given an integer array `nums` of size `n` where `n` is a multiple of `3`. 
You are also given an integer `k` which is a maximum difference between any 3 numbers. 
They want us to divide the array `nums` into arrays of size 3 and put them into a 2D array output. 
If it is not possible to put them into arrays where the difference between any of the 3 numbers is less than `k` then we return an empty array.

Examples:

1. Input: nums = [1,3,4,8,7,9,3,5,1], k = 2
* Output: [[1,1,3],[3,4,5],[7,8,9]]
* Explanation: The difference between any two elements in each array is less than or equal to 2.

2. Input: nums = [2,4,2,2,5,2], k = 2
* Output: []
* Explanation: Because there are four 2s there will be an array with the elements 2 and 5 no matter how we divide it. since 5 - 2 = 3 > k, the condition is not satisfied and so there is no valid division.

It took me a bit, but I did come up with the correct strategy. 
If you sort the input, you can then iterate through three numbers at a time and check if the difference between the 3<sup>rd</sup> number and the 1<sup>st</sup> number is greater than the limit, `k`. 
In my first attempt to do this, I first created a temporary array to push three values to and then determine if I should push it to an output array of arrays of length 3.
This is unnecessary and creates more constant size array memory.
I thought it was necessary because I did not think to iterate 3 numbers at a time and was only iterating one at a time.
i.e., my for loop was iterating using `i++` when I could have gone `i += 3`.
But this isn't even the bottleneck!

I also took the time to write my own quicksort algorithm to sort the numbers, which ended up being incredibly slow. 
I didn't think C++ had a sort method in the standard library and I wanted to write the quicksort algorithm, which I failed at and had to get copilot to explain it to me, so I'll dedicate a section to that. 
However, because the quicksort was so slow I got a time limit exceeded error on the website and I realized that the sort was slow as shit. 
It failed because it was attempting to sort an already sorted array of 10,000,000 number 1s. 
So, I quickly coded a function to check if the array is already sorted.
This revealed that the test suite took over 1 second to complete when quicksort was necessary.
Using the built-in sort function the same test suite only takes 44 ms!
When I first fixed the iteration slowdown, I only saved 16 ms.

### Code
So here's the code for the first iteration of the `divideArray` function which worked to generate the output array the slowest.

```c++
vector<vector<int>> divideArray(vector<int>& nums, int k) {
        // initialize arrays
        vector<vector<int>> out;
        vector<int> tmp = {0,0,0};
        
        // check if already sorted
        if (!sorted(nums)){
            qs(nums);
        }

        // iterate through nums and every 3rd iteration do a check
        for (int i = 0; i < nums.size(); i++) {

            // add into temp until we've filled it
            if (tmp[i % 3] == 0) {
                tmp[i % 3] = nums[i];
            } else { // if it's full, push to out and reset
                out.push_back(tmp);
                tmp = {0,0,0};
                tmp[0] = nums[i];
            }

            // check if the difference is > k, if so return early
            if (tmp[2] - tmp[0] > k){
                return {};
            }
        }
        // the last push isn't done
        out.push_back(tmp);

        return out;
        
    }
```

By changing the for loop to iterate by 3 and just check if the value 2 ahead of the current index causes us to have a difference > k,
we save a non-trivial amount of time. 
This also saves the memory required for a tmp array so those lines get deleted.
This make the suite go from 1035 ms to 1009 ms.

```c++
for (int i = 0; i < nums.size(); i += 3) {

    if (nums[i + 2] - nums[i] > k) {
        return {};
    } // note: making push not in an else runs faster?
    out.push_back({nums[i], nums[i+1], nums[i+2]});
    
}
```

Using the built-in sort saves almost a whole second. 
And we can remove functions for an iterative quicksort and a sorted check.

```c++
sort(nums.begin(), nums.end());
/* Instead of
if (!sorted(nums)){
    qs(nums);
} */
```

## Quicksort
I obviously need to review quicksort, because I failed to implement it the first time from memory.
What is the strategy for quicksort?
You designate a certain element as a pivot and compare all other positions to it.
If an element has a value less than the pivot, it is put in a sublist to the left of the pivot.
If an element has a value greater than the pivot, it is put on the right side.
Then you take the sublists (without the pivot) and choose a pivot again and again, 
until you get sublists which are empty or have one element,
because a list of one item is sorted by definition.
Finally, you put all of the sublists back together around the pivots to get a sorted list.

I specifically wanted to do it iteratively and not recursively, but here I will go over both implementations.
The easier way for me to think about this is actually to do it recursively, because then you don't have to manage pointers to indices.
However, recursion can be slower if the language doesn't have clever tricks.
I know that Python is a language without those tricks, so I've been trying to do things iteratively.
I am not sure if C++ has such limitations, but I would guess that it does because I think it is considered "low-level".
When you do quicksort recursively, you have to call quicksort again on your sublists, which is easier for me to manage.
The basecases are to return up a list that is empty or has one element and the whole list is returned sorted.

### Recursive Quicksort
Here is code to implement a recursive quicksort in C++ on a vector of integers.
It is implemented in a smarter way than described above, making sure not to make copies of the original vector,
sorting it in place without creating an exact copy of the vector.
It works using two functions: `partition()` and `quicksort()`.
The `partition()` function returns the index of the pivot after sorting so that you can call `quicksort()` on the sublists.
Then `quicksort()` has a base case which checks if the low index is greater than or equal to the high index, indicating that you've gone through and sortedthe whole vector.

```c++
#include<vector>
int partition_idx (vector<int>& vec, int lo, int hi) {
    // select the last index as the pivot
    int pivot = vec[hi];
    // To swap values, initialize a pointer to idx before the lo
    int swap_idx = lo - 1;

    // loop through the vector to sort around the pivot
    for (int i = 0; i < hi; i++) {
        if (vec[i] <= pivot) {
            swap_idx++;
            // swap the value at the swap_idx
            int tmp = vec[swap_idx]; // hold value for swap
            vec[swap_idx] = vec[i]; // put current value at swap_idx
            vec[i] = tmp; // put temp value at current idx
        }
    }
    // move the pivot value to the position after the last swap
    swap_idx++;
    vec[hi] = vec[swap_idx];
    vec[swap_idx] = pivot;
    // return the pivot position
    return swap_idx;
}

void quicksort_recursive(vector<int>& vec, int lo, int hi) {
    // base case: check if the idx lo value is greater than the idx hi
    if (lo < hi) {
        // recursion
        // pre step, get pivot (value at last index) and sort vector
        int pivot_idx = partition_idx(vec, lo, hi);

        // recurse step, on sublists not including this iteration's pivot_idx
        quicksort_recursive(vec, lo, pivot_idx - 1);
        quicksort_recursive(vec, pivot_idx + 1, hi);

        // no post step, vector is sorted in place
    }
}
```

### Iterative Quicksort
Instead of recursively calling the function to sort smaller sections of the vector, we can loop through the vector and create our own sort of recursion stack.
Again, we have a separate partition() function and sort in place in exactly the same way.
The quicksort() function is different because we simulate a recursion stack using a stack which is a vector of integer tuples.
We begin by first putting in the indicies for the first and last elements.
While the stack has elements in it, we keep doing the quicksort via the partition() function.


```c++
// exactly the same partition function

void qicksort_iterative(vector<int>& vec) {
    // initialize a stack vector
    vector<pair<int, int>> stack;
    // put in the first & last indicies for initial range
    stack.push_back({0, (int)vec.size() - 1});

    while (!stack.empty()) {
        // assign lo and hi to the last tuple in the stack 
        // (e.g., lo = 0; hi = vec.size() -1)
        auto [lo, hi] = stack.back();
        stack.pop_back(); // actually pop out that tuple

        if (lo < hi) {
            int pivot = partition(vec, lo, hi); // sort around a pivot
            // put the new ranges into the stack not including the pivot
            stack.push_back({lo, pivot - 1});
            stack.push_back({pivot + 1, hi});
        }
    } // the stack gets emptied (line 13) when lo is greater than hi
}
```