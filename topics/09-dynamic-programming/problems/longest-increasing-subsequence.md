# Longest Increasing Subsequence

## 🎯 What Are We Trying To Solve?

Finding the longest increasing subsequence is like finding the longest chain of ascending steps in a staircase where you can skip steps but can't go backwards.

## 📝 The Problem

Given an array of integers, find the length of the longest subsequence where elements are in increasing order (not necessarily consecutive).

```
Example:
Input: [10,9,2,5,3,7,101,18]
Output: 4
Explanation: [2,3,7,101] or [2,3,7,18]
```

## 💻 The Code

```javascript
/**
 * Dynamic Programming Solution
 * Time: O(n²), Space: O(n)
 */
function lengthOfLIS(nums) {
    const n = nums.length;
    const dp = new Array(n).fill(1);
    
    for (let i = 1; i < n; i++) {
        for (let j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
    }
    
    return Math.max(...dp);
}

/**
 * Binary Search Solution (Patience Sorting)
 * Time: O(n log n), Space: O(n)
 */
function lengthOfLISBinarySearch(nums) {
    const tails = [];
    
    for (const num of nums) {
        let left = 0, right = tails.length;
        
        // Find insertion position
        while (left < right) {
            const mid = Math.floor((left + right) / 2);
            if (tails[mid] < num) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        
        tails[left] = num;
    }
    
    return tails.length;
}

/**
 * Get actual LIS sequence
 */
function findLIS(nums) {
    const n = nums.length;
    const dp = new Array(n).fill(1);
    const parent = new Array(n).fill(-1);
    let maxLength = 1;
    let maxIndex = 0;
    
    for (let i = 1; i < n; i++) {
        for (let j = 0; j < i; j++) {
            if (nums[j] < nums[i] && dp[j] + 1 > dp[i]) {
                dp[i] = dp[j] + 1;
                parent[i] = j;
            }
        }
        if (dp[i] > maxLength) {
            maxLength = dp[i];
            maxIndex = i;
        }
    }
    
    // Reconstruct sequence
    const lis = [];
    let curr = maxIndex;
    while (curr !== -1) {
        lis.unshift(nums[curr]);
        curr = parent[curr];
    }
    
    return lis;
}
```

## 🎨 Key Insights

- **DP State**: dp[i] = longest subsequence ending at index i
- **Binary Search**: Maintain array of smallest tail elements
- **Patience Sorting**: Card game analogy for understanding

## ⏱️ Complexity

- DP: O(n²) time, O(n) space
- Binary Search: O(n log n) time, O(n) space
