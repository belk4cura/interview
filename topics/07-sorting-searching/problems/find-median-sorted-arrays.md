# Median of Two Sorted Arrays

## 🎯 What Are We Trying To Solve?

Finding the median of two sorted arrays is like merging two sorted lines of people by height and finding the middle person - but we want to do it without actually merging them.

## 📝 The Problem

Given two sorted arrays, find the median of the combined sorted array in O(log(min(m,n))) time.

```
Example:
nums1 = [1, 3], nums2 = [2]
Output: 2.0

nums1 = [1, 2], nums2 = [3, 4]
Output: 2.5 (average of 2 and 3)
```

## 💻 The Code

```javascript
/**
 * Binary Search Solution
 * Time: O(log(min(m,n))), Space: O(1)
 */
function findMedianSortedArrays(nums1, nums2) {
    // Ensure nums1 is the smaller array
    if (nums1.length > nums2.length) {
        return findMedianSortedArrays(nums2, nums1);
    }
    
    const m = nums1.length;
    const n = nums2.length;
    let low = 0, high = m;
    
    while (low <= high) {
        const partitionX = Math.floor((low + high) / 2);
        const partitionY = Math.floor((m + n + 1) / 2) - partitionX;
        
        const maxLeftX = partitionX === 0 ? -Infinity : nums1[partitionX - 1];
        const minRightX = partitionX === m ? Infinity : nums1[partitionX];
        
        const maxLeftY = partitionY === 0 ? -Infinity : nums2[partitionY - 1];
        const minRightY = partitionY === n ? Infinity : nums2[partitionY];
        
        if (maxLeftX <= minRightY && maxLeftY <= minRightX) {
            // Found correct partition
            if ((m + n) % 2 === 0) {
                return (Math.max(maxLeftX, maxLeftY) + 
                        Math.min(minRightX, minRightY)) / 2;
            } else {
                return Math.max(maxLeftX, maxLeftY);
            }
        } else if (maxLeftX > minRightY) {
            high = partitionX - 1;
        } else {
            low = partitionX + 1;
        }
    }
    
    throw new Error("Arrays not sorted");
}

/**
 * Simple Merge Solution (not optimal but intuitive)
 * Time: O(m+n), Space: O(1)
 */
function findMedianSimple(nums1, nums2) {
    const m = nums1.length, n = nums2.length;
    const total = m + n;
    let i = 0, j = 0;
    let prev = 0, curr = 0;
    
    for (let count = 0; count <= Math.floor(total / 2); count++) {
        prev = curr;
        
        if (i < m && (j >= n || nums1[i] <= nums2[j])) {
            curr = nums1[i++];
        } else {
            curr = nums2[j++];
        }
    }
    
    return total % 2 === 0 ? (prev + curr) / 2 : curr;
}
```

## 🎨 Key Insights

- **Binary Search on Partition**: Find where to split arrays
- **Partition Properties**: Left elements ≤ right elements
- **Edge Cases**: Handle empty arrays and boundaries

## ⏱️ Complexity

- Binary Search: O(log(min(m,n))) time, O(1) space
- Merge: O(m+n) time, O(1) space
