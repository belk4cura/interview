# Median of Two Sorted Arrays

## 🎯 What Are We Trying To Solve?

Imagine you have two sorted lists of test scores from different classes, and you want to find the overall median score WITHOUT merging the lists. It's like finding the middle value of a combined deck of cards without actually shuffling them together!

## 📝 The Problem

Find the median of two sorted arrays. The overall run time complexity should be O(log(min(m,n))).

```
Example 1:
nums1 = [1, 3], nums2 = [2]
Output: 2.0 (The median is 2)

Example 2:
nums1 = [1, 2], nums2 = [3, 4]
Output: 2.5 (The median is (2 + 3) / 2 = 2.5)
```

## 🧠 How to Think About This

The key insight: We need to partition both arrays such that:
1. Left partition has same number of elements as right partition (±1)
2. All elements in left partition ≤ all elements in right partition

We binary search for the correct partition point!

### Visual Process
```
nums1: [1, 3, 8, 9]      
nums2: [2, 4, 5, 6, 7]

Find partition where:
Left side: [1, 2, 3, 4] | Right side: [5, 6, 7, 8, 9]
                    ↑ Median = (4 + 5) / 2 = 4.5
```

## 💻 The Code

```javascript
/**
 * Find median of two sorted arrays
 * Time: O(log min(m, n)) - binary search on smaller array
 * Space: O(1) - only using pointers
 */
function findMedianSortedArrays(nums1, nums2) {
    // Ensure nums1 is the smaller array for efficiency
    // Why? Binary search on smaller array = fewer iterations
    if (nums1.length > nums2.length) {
        return findMedianSortedArrays(nums2, nums1);
    }
    
    const m = nums1.length;
    const n = nums2.length;
    let low = 0;
    let high = m;
    
    // Binary search for correct partition
    while (low <= high) {
        // Partition nums1 at partitionX
        const partitionX = Math.floor((low + high) / 2);
        
        // Calculate corresponding partition for nums2
        // +1 handles odd total length (left side gets extra element)
        const partitionY = Math.floor((m + n + 1) / 2) - partitionX;
        
        // Find elements at partition boundaries
        // Use -Infinity and Infinity for edge cases
        const maxLeftX = partitionX === 0 ? -Infinity : nums1[partitionX - 1];
        const minRightX = partitionX === m ? Infinity : nums1[partitionX];
        
        const maxLeftY = partitionY === 0 ? -Infinity : nums2[partitionY - 1];
        const minRightY = partitionY === n ? Infinity : nums2[partitionY];
        
        // Check if we found valid partition
        if (maxLeftX <= minRightY && maxLeftY <= minRightX) {
            // Found correct partition!
            
            if ((m + n) % 2 === 0) {
                // Even total length: average of two middle elements
                return (Math.max(maxLeftX, maxLeftY) + 
                        Math.min(minRightX, minRightY)) / 2;
            } else {
                // Odd total length: middle element (max of left side)
                return Math.max(maxLeftX, maxLeftY);
            }
        } else if (maxLeftX > minRightY) {
            // nums1's partition is too far right
            // Move left in nums1
            high = partitionX - 1;
        } else {
            // nums1's partition is too far left
            // Move right in nums1
            low = partitionX + 1;
        }
    }
    
    throw new Error("Arrays not sorted");
}

// Test cases
console.log(findMedianSortedArrays([1, 3], [2]));           // 2
console.log(findMedianSortedArrays([1, 2], [3, 4]));        // 2.5
console.log(findMedianSortedArrays([0, 0], [0, 0]));        // 0
console.log(findMedianSortedArrays([], [1]));               // 1
console.log(findMedianSortedArrays([2], []));               // 2
```

## 🎨 Step-by-Step Walkthrough

Let's find the median of [1, 3] and [2]:

```
nums1 = [1, 3], nums2 = [2]
Total length = 3 (odd), median at position 1

Initial: low = 0, high = 2 (m = 2)

Iteration 1:
partitionX = (0 + 2) / 2 = 1
partitionY = (2 + 1 + 1) / 2 - 1 = 1

nums1: [1 | 3]     Left of partition: nums1[0] = 1
nums2: [2 |  ]     Left of partition: nums2[0] = 2

maxLeftX = 1, minRightX = 3
maxLeftY = 2, minRightY = Infinity

Check: 1 ≤ Infinity ✓ and 2 ≤ 3 ✓
Valid partition found!

Odd length → median = max(1, 2) = 2 ✓
```

Another example with [1, 2] and [3, 4]:

```
nums1 = [1, 2], nums2 = [3, 4]
Total length = 4 (even), median is average of positions 1 and 2

Iteration 1:
partitionX = 1
partitionY = (2 + 2 + 1) / 2 - 1 = 1

nums1: [1 | 2]     maxLeftX = 1, minRightX = 2
nums2: [3 | 4]     maxLeftY = 3, minRightY = 4

Check: 1 ≤ 4 ✓ but 3 > 2 ✗
maxLeftY > minRightX, so move right in nums1

Iteration 2:
partitionX = 2
partitionY = 0

nums1: [1, 2 | ]   maxLeftX = 2, minRightX = Infinity
nums2: [ | 3, 4]   maxLeftY = -Infinity, minRightY = 3

Check: 2 ≤ 3 ✓ and -Infinity ≤ Infinity ✓
Valid partition!

Even length → median = (max(2, -∞) + min(∞, 3)) / 2 = (2 + 3) / 2 = 2.5 ✓
```

## 📊 Why O(log min(m,n))?

### Binary Search Analysis

```
We binary search on the smaller array:
- Start with search space of size min(m, n)
- Each iteration halves the search space
- Total iterations: log(min(m, n))

Each iteration does O(1) work:
- Calculate partitions
- Compare 4 elements
- No array traversal!
```

## 🚫 Common Mistakes

1. **Not Handling Edge Cases**
   ```javascript
   // Wrong: Array access without bounds check
   const maxLeftX = nums1[partitionX - 1];  // Crashes if partitionX = 0
   
   // Right: Use infinity for edge cases
   const maxLeftX = partitionX === 0 ? -Infinity : nums1[partitionX - 1];
   ```

2. **Wrong Partition Calculation**
   ```javascript
   // Wrong: Doesn't handle odd length properly
   const partitionY = (m + n) / 2 - partitionX;
   
   // Right: +1 ensures left side gets extra element for odd total
   const partitionY = Math.floor((m + n + 1) / 2) - partitionX;
   ```

3. **Not Ensuring Smaller Array First**
   ```javascript
   // Wrong: Might binary search on larger array
   // This could cause partitionY to be negative!
   
   // Right: Always search on smaller array
   if (nums1.length > nums2.length) {
       return findMedianSortedArrays(nums2, nums1);
   }
   ```

## 🔄 Alternative Approaches

### Approach 2: Merge Arrays (Simple but O(m+n))
```javascript
function findMedianMerge(nums1, nums2) {
    const merged = [];
    let i = 0, j = 0;
    
    // Merge arrays
    while (i < nums1.length && j < nums2.length) {
        if (nums1[i] <= nums2[j]) {
            merged.push(nums1[i++]);
        } else {
            merged.push(nums2[j++]);
        }
    }
    
    while (i < nums1.length) merged.push(nums1[i++]);
    while (j < nums2.length) merged.push(nums2[j++]);
    
    // Find median
    const mid = Math.floor(merged.length / 2);
    if (merged.length % 2 === 0) {
        return (merged[mid - 1] + merged[mid]) / 2;
    } else {
        return merged[mid];
    }
}
// Time: O(m + n), Space: O(m + n)
```

### Approach 3: Two Pointers (O(m+n) time, O(1) space)
```javascript
function findMedianTwoPointers(nums1, nums2) {
    const total = nums1.length + nums2.length;
    const mid = Math.floor(total / 2);
    let i = 0, j = 0;
    let prev = 0, curr = 0;
    
    for (let count = 0; count <= mid; count++) {
        prev = curr;
        
        if (i >= nums1.length) {
            curr = nums2[j++];
        } else if (j >= nums2.length) {
            curr = nums1[i++];
        } else if (nums1[i] <= nums2[j]) {
            curr = nums1[i++];
        } else {
            curr = nums2[j++];
        }
    }
    
    if (total % 2 === 0) {
        return (prev + curr) / 2;
    } else {
        return curr;
    }
}
// Time: O(m + n), Space: O(1)
```

## 🔗 Related Problems

- **Kth Element in Two Sorted Arrays** - Generalization of this problem
- **Median of Data Stream** - Dynamic median calculation
- **Find K-th Smallest Pair Distance** - Similar binary search technique
- **Split Array Largest Sum** - Binary search on answer

## 💡 Key Takeaways

1. **Binary Search on Partition**: Not searching for a value, but a position
2. **Mathematical Insight**: Partitioning defines the median
3. **Edge Case Handling**: Infinity values simplify boundary logic
4. **Optimize Search Space**: Always search on smaller array

This is a classic hard problem that tests understanding of binary search beyond simple value searching!
