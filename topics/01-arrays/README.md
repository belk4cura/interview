# Arrays

## 📚 Overview

Arrays are the most fundamental data structure in computer science. They store elements in contiguous memory locations, allowing for constant-time access by index.

## 🎯 What You Need to Know

### Key Properties
- **Fixed Size**: Arrays have a predetermined size (though dynamic arrays can resize)
- **Indexed Access**: O(1) time to access any element by position
- **Contiguous Memory**: Elements stored next to each other in memory
- **Cache Friendly**: Sequential access is very fast due to CPU cache

### Time Complexities
| Operation | Time | Why |
|-----------|------|-----|
| Access by index | O(1) | Direct memory calculation |
| Search (unsorted) | O(n) | Must check each element |
| Search (sorted) | O(log n) | Binary search |
| Insert at end | O(1)* | Just add to end |
| Insert at beginning | O(n) | Shift all elements |
| Delete | O(n) | Shift elements to fill gap |

*Amortized for dynamic arrays

### Space Complexity
- O(n) for storing n elements
- Dynamic arrays may have up to 2n capacity

## 🔑 Essential Patterns

### 1. Two Pointers
Use two pointers moving toward each other or in the same direction.

```javascript
// Example: Check if array is palindrome
function isPalindrome(arr) {
    let left = 0;                    // Start at beginning
    let right = arr.length - 1;      // Start at end
    
    while (left < right) {           // Move pointers toward center
        if (arr[left] !== arr[right]) return false;  // Mismatch = not palindrome
        left++;      // Move left pointer right →
        right--;     // Move right pointer left ←
    }
    return true;     // All matched = palindrome!
}
```

**Visual Example - Palindrome:**
```
isPalindrome([1, 2, 3, 2, 1])

Step 1:  [1, 2, 3, 2, 1]
          ↑           ↑
        left        right
        1 === 1 ✓ Continue

Step 2:  [1, 2, 3, 2, 1]
             ↑     ↑
           left  right
           2 === 2 ✓ Continue

Step 3:  [1, 2, 3, 2, 1]
                ↑
            left/right meet
            Done! Return true
```

**Visual Example - Non-Palindrome:**
```
isPalindrome([1, 2, 3, 4, 5])

Step 1:  [1, 2, 3, 4, 5]
          ↑           ↑
        left        right
        1 !== 5 ✗ Return false immediately!
```

### 2. Sliding Window
Maintain a window of elements for substring/subarray problems.

```javascript
// Example: Maximum sum of k consecutive elements
function maxSumSubarray(arr, k) {
    let sum = 0, maxSum = 0;
    
    // Build initial window of size k
    for (let i = 0; i < k; i++) {
        sum += arr[i];  // Add first k elements
    }
    maxSum = sum;  // First window sum is our initial max
    
    // Slide the window: remove leftmost, add rightmost
    for (let i = k; i < arr.length; i++) {
        sum = sum - arr[i - k] + arr[i];  // Remove left, add right
        maxSum = Math.max(maxSum, sum);   // Update max if needed
    }
    
    return maxSum;
}
```

**Visual Example:** `maxSumSubarray([2, 1, 5, 1, 3, 2], k=3)`
```
Initial Window (k=3):
[2, 1, 5, 1, 3, 2]
 └──┬──┘
   sum = 2+1+5 = 8, maxSum = 8

Slide 1: Remove 2, Add 1
[2, 1, 5, 1, 3, 2]
    └──┬──┘
   sum = 8-2+1 = 7, maxSum = 8

Slide 2: Remove 1, Add 3
[2, 1, 5, 1, 3, 2]
       └──┬──┘
   sum = 7-1+3 = 9, maxSum = 9 ← New max!

Slide 3: Remove 5, Add 2
[2, 1, 5, 1, 3, 2]
          └──┬──┘
   sum = 9-5+2 = 6, maxSum = 9

Result: 9
```

### 3. Prefix Sums
Precompute cumulative sums for range queries.

```javascript
// Build prefix sum array
function buildPrefixSum(arr) {
    const prefix = [0];  // Start with 0 for easier calculation
    
    // Each element stores sum of all elements up to that index
    for (let i = 0; i < arr.length; i++) {
        prefix[i + 1] = prefix[i] + arr[i];  // Add current element to previous sum
    }
    
    return prefix;
}

// Get sum from index i to j in O(1) time
// Formula: prefix[j+1] - prefix[i]
```

**Visual Example:** `buildPrefixSum([3, 1, 4, 1, 5])`
```
Original:  [3, 1, 4, 1, 5]
Index:      0  1  2  3  4

Building Prefix Array:
prefix[0] = 0                    (base case)
prefix[1] = 0 + 3 = 3           (sum up to index 0)
prefix[2] = 3 + 1 = 4           (sum up to index 1)
prefix[3] = 4 + 4 = 8           (sum up to index 2)
prefix[4] = 8 + 1 = 9           (sum up to index 3)
prefix[5] = 9 + 5 = 14          (sum up to index 4)

Result: [0, 3, 4, 8, 9, 14]

Range Query Examples:
Sum from index 1 to 3: prefix[4] - prefix[1] = 9 - 3 = 6
  → arr[1] + arr[2] + arr[3] = 1 + 4 + 1 = 6 ✓

Sum from index 0 to 4: prefix[5] - prefix[0] = 14 - 0 = 14
  → arr[0] + arr[1] + arr[2] + arr[3] + arr[4] = 3+1+4+1+5 = 14 ✓
```

### 4. Kadane's Algorithm
Find maximum subarray sum.

```javascript
function maxSubarraySum(arr) {
    let maxSoFar = arr[0];        // Best sum found so far (global max)
    let maxEndingHere = arr[0];   // Best sum ending at current position (local max)
    
    for (let i = 1; i < arr.length; i++) {
        // Either extend existing subarray or start fresh from current element
        maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
        
        // Update global maximum if current subarray is better
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }
    
    return maxSoFar;
}
```

**Visual Example:** `maxSubarraySum([-2, 1, -3, 4, -1, 2, 1, -5, 4])`
```
Index:     0   1   2   3   4   5   6   7   8
Array:   [-2,  1, -3,  4, -1,  2,  1, -5,  4]

Step-by-step trace:
i=0: maxEndingHere = -2, maxSoFar = -2

i=1: maxEndingHere = max(1, -2+1) = max(1, -1) = 1
     maxSoFar = max(-2, 1) = 1
     Decision: Start fresh at 1 (better than extending -2)

i=2: maxEndingHere = max(-3, 1-3) = max(-3, -2) = -2
     maxSoFar = max(1, -2) = 1
     Decision: Extend from 1 (gives -2)

i=3: maxEndingHere = max(4, -2+4) = max(4, 2) = 4
     maxSoFar = max(1, 4) = 4
     Decision: Start fresh at 4 (better than extending -2)

i=4: maxEndingHere = max(-1, 4-1) = max(-1, 3) = 3
     maxSoFar = max(4, 3) = 4
     Decision: Extend from 4 (gives 3)

i=5: maxEndingHere = max(2, 3+2) = max(2, 5) = 5
     maxSoFar = max(4, 5) = 5
     Decision: Extend (gives 5)

i=6: maxEndingHere = max(1, 5+1) = max(1, 6) = 6
     maxSoFar = max(5, 6) = 6 ← New maximum!
     Decision: Extend (gives 6)

i=7: maxEndingHere = max(-5, 6-5) = max(-5, 1) = 1
     maxSoFar = max(6, 1) = 6
     Decision: Extend (gives 1)

i=8: maxEndingHere = max(4, 1+4) = max(4, 5) = 5
     maxSoFar = max(6, 5) = 6
     Decision: Extend (gives 5)

Result: 6 (subarray [4, -1, 2, 1])
```

## 💡 JavaScript Array Methods

### Mutating Methods (Change Original)
```javascript
arr.push(item)      // Add to end - O(1)
arr.pop()           // Remove from end - O(1)
arr.unshift(item)   // Add to beginning - O(n)
arr.shift()         // Remove from beginning - O(n)
arr.splice(i, n)    // Remove n items at index i - O(n)
arr.sort()          // Sort in place - O(n log n)
arr.reverse()       // Reverse in place - O(n)
```

### Non-Mutating Methods (Return New)
```javascript
arr.slice(start, end)  // Copy portion - O(n)
arr.concat(arr2)       // Combine arrays - O(n + m)
arr.map(fn)            // Transform elements - O(n)
arr.filter(fn)         // Filter elements - O(n)
arr.reduce(fn, init)   // Reduce to single value - O(n)
```

### Search Methods
```javascript
arr.indexOf(item)      // First index of item - O(n)
arr.includes(item)     // Check if exists - O(n)
arr.find(fn)          // First element matching - O(n)
arr.findIndex(fn)     // Index of first match - O(n)
```

## 🎯 Common Interview Patterns

### Pattern 1: Hash Map for O(1) Lookup
```javascript
// Two Sum problem
function twoSum(arr, target) {
    const seen = new Map();
    for (let i = 0; i < arr.length; i++) {
        const complement = target - arr[i];
        if (seen.has(complement)) {
            return [seen.get(complement), i];
        }
        seen.set(arr[i], i);
    }
    return [];
}
```

### Pattern 2: Sorting First
```javascript
// Find pairs with given sum (all pairs)
function findPairs(arr, target) {
    arr.sort((a, b) => a - b);
    let left = 0, right = arr.length - 1;
    const pairs = [];
    
    while (left < right) {
        const sum = arr[left] + arr[right];
        if (sum === target) {
            pairs.push([arr[left], arr[right]]);
            left++;
            right--;
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    return pairs;
}
```

### Pattern 3: Multiple Passes
```javascript
// Product of array except self
function productExceptSelf(nums) {
    const n = nums.length;
    const result = new Array(n).fill(1);
    
    // Left pass
    for (let i = 1; i < n; i++) {
        result[i] = result[i - 1] * nums[i - 1];
    }
    
    // Right pass
    let rightProduct = 1;
    for (let i = n - 1; i >= 0; i--) {
        result[i] *= rightProduct;
        rightProduct *= nums[i];
    }
    
    return result;
}
```

## 🚫 Common Pitfalls

1. **Off-by-one errors**
   - Check array bounds carefully
   - Remember arrays are 0-indexed
   
2. **Modifying array while iterating**
   - Can cause skipped elements or infinite loops
   - Use backward iteration or create a copy

3. **Not handling empty arrays**
   - Always check for `arr.length === 0`

4. **Integer overflow**
   - JavaScript handles this automatically but be aware for other languages

5. **Reference vs Value**
   ```javascript
   const arr1 = [1, 2, 3];
   const arr2 = arr1;  // Reference, not copy!
   arr2[0] = 99;       // arr1 is now [99, 2, 3]
   
   // To copy:
   const arr3 = [...arr1];  // Shallow copy
   const arr4 = arr1.slice(); // Also shallow copy
   ```

## 📝 Practice Problems in This Section

### Two-Pointer Techniques
1. [Two Sum](./problems/two-sum.md) - Classic hash map pattern ⭐
2. [Three Sum](./problems/three-sum.md) - Extended two-pointer problem ⭐
3. [Container With Most Water](./problems/container-with-most-water.md) - Maximize area ⭐
4. [Remove Duplicates from Sorted Array](./problems/remove-duplicates-sorted.md) - In-place modification
5. [Valid Palindrome II](./problems/valid-palindrome-ii.md) - String manipulation with deletion
6. [Move Zeroes](./problems/move-zeroes.md) - In-place partitioning

### Advanced Techniques
7. [Sliding Window Maximum](./problems/sliding-window-maximum.md) - Deque optimization ⭐
8. [Trapping Rain Water](./problems/trapping-rain-water.md) - Complex two-pointer ⭐

### Fundamental Problems
9. [A Very Big Sum](./problems/very-big-sum.md) - Handle large numbers
10. [Designer PDF Viewer](./problems/designer-pdf-viewer.md) - Array manipulation
11. [Left Rotation](./problems/left-rotation.md) - Array rotation techniques
12. [Sparse Arrays](./problems/sparse-arrays.md) - Frequency counting

## 🎯 Mastery Checklist

- [ ] Can implement dynamic array from scratch
- [ ] Understand amortized time complexity
- [ ] Know when to use two pointers vs sliding window
- [ ] Can identify when sorting helps solve a problem
- [ ] Comfortable with all JavaScript array methods
- [ ] Can optimize space complexity when needed
- [ ] Understand cache locality and why arrays are fast

## 🔗 Next Steps

Once you've mastered arrays, move on to [Linked Lists](../02-linked-lists/README.md) to understand the trade-offs between contiguous and non-contiguous storage.
