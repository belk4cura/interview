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
    let left = 0, right = arr.length - 1;
    while (left < right) {
        if (arr[left] !== arr[right]) return false;
        left++;
        right--;
    }
    return true;
}
```

### 2. Sliding Window
Maintain a window of elements for substring/subarray problems.
```javascript
// Example: Maximum sum of k consecutive elements
function maxSumSubarray(arr, k) {
    let sum = 0, maxSum = 0;
    // Initial window
    for (let i = 0; i < k; i++) sum += arr[i];
    maxSum = sum;
    // Slide window
    for (let i = k; i < arr.length; i++) {
        sum = sum - arr[i - k] + arr[i];
        maxSum = Math.max(maxSum, sum);
    }
    return maxSum;
}
```

### 3. Prefix Sums
Precompute cumulative sums for range queries.
```javascript
// Build prefix sum array
function buildPrefixSum(arr) {
    const prefix = [0];
    for (let i = 0; i < arr.length; i++) {
        prefix[i + 1] = prefix[i] + arr[i];
    }
    return prefix;
}
// Sum from index i to j: prefix[j+1] - prefix[i]
```

### 4. Kadane's Algorithm
Find maximum subarray sum.
```javascript
function maxSubarraySum(arr) {
    let maxSoFar = arr[0];
    let maxEndingHere = arr[0];
    
    for (let i = 1; i < arr.length; i++) {
        maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }
    
    return maxSoFar;
}
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

1. [Two Sum](./problems/two-sum.md) - Classic hash map pattern
2. [A Very Big Sum](./problems/very-big-sum.md) - Handle large numbers
3. [Designer PDF Viewer](./problems/designer-pdf-viewer.md) - Array manipulation
4. [Left Rotation](./problems/left-rotation.md) - Array rotation techniques
5. [Sparse Arrays](./problems/sparse-arrays.md) - Frequency counting

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
