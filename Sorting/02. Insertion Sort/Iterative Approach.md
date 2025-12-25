## Insertion Sort Iterative Approach

## Intuition (Insertion Sort)
Insertion sort works like sorting playing cards in your hand:
- You keep the left part of the array **sorted**.
- Pick the next element (`key`) from the unsorted part.
- Shift all bigger elements in the sorted part one step to the right.
- Insert `key` into the correct position.

After each step `i`, the subarray `nums[0..i]` becomes sorted.

## Algorithm Steps
1. Start from the second element (`i = 1`) because a single element is already sorted.
2. Set `key = nums[i]` (the element to insert into the sorted left part).
3. Set `j = i - 1` and compare `key` with elements to its left:
   - While `j >= 0` and `nums[j] > key`:
     - shift `nums[j]` to `nums[j+1]`
     - decrement `j`
4. Place `key` at `nums[j+1]` (its correct position).
5. Repeat for all `i` from `1` to `n-1`.
6. Return the sorted array.


``` cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    // Function to sort the array using insertion sort
    vector<int> insertionSort(vector<int>& nums) {
        int n = nums.size(); // Size of the array 
        
        // For every element in the array 
        for (int i = 1; i < n; i++) {
            int key = nums[i]; // Current element as key 
            int j = i - 1; 
            
            // Shift elements that are greater than key by one position
            while (j >= 0 && nums[j] > key) {
                nums[j + 1] = nums[j];
                j--;
            }
            
            nums[j + 1] = key; // Insert key at correct position
        }
        
        return nums;
    }
};


int main() {
    // Create an instance of the solution class
    Solution solution;
    
    vector<int> nums = {13, 46, 24, 52, 20, 9};
    
    cout << "Before Using Insertion Sort: " << endl;
    for (int num : nums) {
        cout << num << " ";
    }
    cout << endl;

    // Function call for insertion sort
    nums = solution.insertionSort(nums);

    cout << "After Using Insertion Sort: " << endl;
    for (int num : nums) {
        cout << num << " ";
    }
    cout << endl;

    return 0;
}
```

## Complexity Analysis (Insertion Sort)

Let `n` be the number of elements.

Insertion sort cost depends on how many **shifts** are needed.
A shift happens when a previous element is greater than `key`.

---

### 1) Best Case — Already Sorted
**Example:** `[1, 2, 3, 4, 5]`

- For every `i`, `nums[j] > key` is false immediately.
- Inner `while` loop runs 0 times (no shifting).

**Time:** `O(n)`  
- Comparisons: about `n-1`
- Shifts: `0` (only key placed back in same spot)

**Space:** `O(1)`

---

### 2) Worst Case — Reverse Sorted
**Example:** `[5, 4, 3, 2, 1]`

- For each `i`, `key` is smaller than all elements in `nums[0..i-1]`.
- Inner loop shifts all `i` elements to the right.

Total shifts/comparisons:
\[
1 + 2 + 3 + \dots + (n-1) = \frac{n(n-1)}{2}
\]

**Time:** `O(n^2)`  
- Comparisons: `n(n-1)/2`
- Shifts: `n(n-1)/2`

**Space:** `O(1)`

---

### 3) Average Case — Random Order
**Example:** `[13, 46, 24, 52, 20, 9]`

- On average, each `key` moves about halfway back into the sorted part.
- Expected shifts per element ≈ `i/2`.

Total expected work:
\[
\sum_{i=1}^{n-1} \frac{i}{2} = O(n^2)
\]

**Time:** `O(n^2)` (average)  
**Space:** `O(1)`

---

## Summary Table

| Case    | Example            | Time     | Why |
|---------|---------------------|----------|-----|
| Best    | `[1,2,3,4,5]`       | `O(n)`   | almost no shifting |
| Average | random order        | `O(n^2)` | moderate shifting |
| Worst   | `[5,4,3,2,1]`       | `O(n^2)` | maximum shifting |

**Extra Space:** `O(1)` in all cases.
