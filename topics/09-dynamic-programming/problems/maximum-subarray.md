# Maximum Subarray

## 🎯 What Are We Trying To Solve?

Find the contiguous subarray with the maximum sum in an array of integers (Kadane's Algorithm).

## 📝 The Problem

Given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

```
Example 1:
Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6

Example 2:
Input: nums = [1]
Output: 1

Example 3:
Input: nums = [5,4,-1,7,8]
Output: 23
Explanation: Entire array has max sum
```

## 💻 The Code

```javascript
/**
 * Maximum Subarray - Kadane's Algorithm
 * Time: O(n), Space: O(1)
 */
function maxSubArray(nums) {
    let maxCurrent = nums[0];
    let maxGlobal = nums[0];
    
    for (let i = 1; i < nums.length; i++) {
        // Either extend current subarray or start new one
        maxCurrent = Math.max(nums[i], maxCurrent + nums[i]);
        // Update global maximum
        maxGlobal = Math.max(maxGlobal, maxCurrent);
    }
    
    return maxGlobal;
}

/**
 * Dynamic Programming approach (explicit)
 * Time: O(n), Space: O(n)
 */
function maxSubArrayDP(nums) {
    const n = nums.length;
    const dp = new Array(n);
    dp[0] = nums[0];
    let maxSum = nums[0];
    
    for (let i = 1; i < n; i++) {
        // dp[i] = max sum ending at index i
        dp[i] = Math.max(nums[i], dp[i - 1] + nums[i]);
        maxSum = Math.max(maxSum, dp[i]);
    }
    
    return maxSum;
}

/**
 * Get actual subarray indices
 * Time: O(n), Space: O(1)
 */
function maxSubArrayWithIndices(nums) {
    let maxCurrent = nums[0];
    let maxGlobal = nums[0];
    let start = 0, end = 0, tempStart = 0;
    
    for (let i = 1; i < nums.length; i++) {
        // Start new subarray if current element is larger
        if (nums[i] > maxCurrent + nums[i]) {
            maxCurrent = nums[i];
            tempStart = i;
        } else {
            maxCurrent = maxCurrent + nums[i];
        }
        
        // Update global maximum and indices
        if (maxCurrent > maxGlobal) {
            maxGlobal = maxCurrent;
            start = tempStart;
            end = i;
        }
    }
    
    return {
        sum: maxGlobal,
        start: start,
        end: end,
        subarray: nums.slice(start, end + 1)
    };
}

/**
 * Divide and Conquer approach
 * Time: O(n log n), Space: O(log n)
 */
function maxSubArrayDivideConquer(nums) {
    function helper(left, right) {
        if (left > right) return -Infinity;
        
        const mid = Math.floor((left + right) / 2);
        let leftSum = 0, rightSum = 0;
        let currSum = 0;
        
        // Find max sum for left half including mid
        for (let i = mid - 1; i >= left; i--) {
            currSum += nums[i];
            leftSum = Math.max(leftSum, currSum);
        }
        
        // Find max sum for right half including mid
        currSum = 0;
        for (let i = mid + 1; i <= right; i++) {
            currSum += nums[i];
            rightSum = Math.max(rightSum, currSum);
        }
        
        // Max of: left half, right half, or crossing mid
        const crossSum = nums[mid] + leftSum + rightSum;
        const leftMax = helper(left, mid - 1);
        const rightMax = helper(mid + 1, right);
        
        return Math.max(crossSum, leftMax, rightMax);
    }
    
    return helper(0, nums.length - 1);
}

/**
 * Maximum Product Subarray
 * Time: O(n), Space: O(1)
 */
function maxProduct(nums) {
    let maxProduct = nums[0];
    let minProduct = nums[0];
    let result = nums[0];
    
    for (let i = 1; i < nums.length; i++) {
        // When negative, max and min swap
        if (nums[i] < 0) {
            [maxProduct, minProduct] = [minProduct, maxProduct];
        }
        
        maxProduct = Math.max(nums[i], maxProduct * nums[i]);
        minProduct = Math.min(nums[i], minProduct * nums[i]);
        
        result = Math.max(result, maxProduct);
    }
    
    return result;
}

/**
 * Maximum Subarray Sum Circular
 * Time: O(n), Space: O(1)
 */
function maxSubarraySumCircular(nums) {
    let maxKadane = kadane(nums);
    let totalSum = 0;
    
    for (let i = 0; i < nums.length; i++) {
        totalSum += nums[i];
        nums[i] = -nums[i]; // Invert array
    }
    
    let minKadane = kadane(nums);
    
    // Restore original array
    for (let i = 0; i < nums.length; i++) {
        nums[i] = -nums[i];
    }
    
    // Max is either regular max or total - min
    if (totalSum + minKadane === 0) {
        return maxKadane; // All negative numbers
    }
    
    return Math.max(maxKadane, totalSum + minKadane);
}

function kadane(nums) {
    let maxCurrent = nums[0];
    let maxGlobal = nums[0];
    
    for (let i = 1; i < nums.length; i++) {
        maxCurrent = Math.max(nums[i], maxCurrent + nums[i]);
        maxGlobal = Math.max(maxGlobal, maxCurrent);
    }
    
    return maxGlobal;
}

// Test cases
console.log(maxSubArray([-2,1,-3,4,-1,2,1,-5,4])); // 6
console.log(maxSubArray([1])); // 1
console.log(maxSubArray([5,4,-1,7,8])); // 23
console.log(maxSubArray([-1])); // -1
console.log(maxSubArray([-2,-1])); // -1

const result = maxSubArrayWithIndices([-2,1,-3,4,-1,2,1,-5,4]);
console.log(result); // {sum: 6, start: 3, end: 6, subarray: [4,-1,2,1]}

console.log(maxProduct([2,3,-2,4])); // 6
console.log(maxProduct([-2,0,-1])); // 0
console.log(maxSubarraySumCircular([1,-2,3,-2])); // 3
console.log(maxSubarraySumCircular([5,-3,5])); // 10
```

## 🎨 The Approach

### Kadane's Algorithm:
1. Track maximum sum ending at current position
2. At each element, choose: start new subarray or extend current
3. Keep global maximum

### Visual Walkthrough:
```
Array: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

i=0: maxCurrent = -2, maxGlobal = -2
i=1: maxCurrent = max(1, -2+1) = 1, maxGlobal = 1
i=2: maxCurrent = max(-3, 1-3) = -2, maxGlobal = 1
i=3: maxCurrent = max(4, -2+4) = 4, maxGlobal = 4
i=4: maxCurrent = max(-1, 4-1) = 3, maxGlobal = 4
i=5: maxCurrent = max(2, 3+2) = 5, maxGlobal = 5
i=6: maxCurrent = max(1, 5+1) = 6, maxGlobal = 6
i=7: maxCurrent = max(-5, 6-5) = 1, maxGlobal = 6
i=8: maxCurrent = max(4, 1+4) = 5, maxGlobal = 6

Result: 6 (subarray [4,-1,2,1])
```

## 🤔 Why This Works

- **Local vs Global**: Track best ending here vs best overall
- **Decision at each step**: Include current element or start fresh
- **Negative numbers**: Can be included if overall sum increases
- **Optimal substructure**: Max sum ending at i depends on max sum ending at i-1

## ⚠️ Common Pitfalls

1. **All negative numbers** - Must return least negative
2. **Empty subarray** - Problem requires at least one element
3. **Integer overflow** - Large sums possible
4. **Starting position** - Don't assume starts at index 0

## 🎯 Variations

### Maximum Product Subarray
- Track both max and min (negatives can flip)
- Handle zeros specially

### Circular Array
- Consider wrap-around
- Max is either normal or circular

### K Subarrays
- Find k non-overlapping subarrays with max sum

### 2D Maximum Subarray
- Apply to matrix (harder problem)

## 📊 Complexity Analysis

- **Time**: O(n) - Single pass for Kadane's
- **Space**: O(1) - Only tracking current and max
- **Divide & Conquer**: O(n log n) time
- **Optimal**: Can't do better than O(n)

## 🔄 Related Problems

- **Maximum Product Subarray** - Multiplication variant
- **Maximum Sum Circular Subarray** - Circular array
- **Maximum Average Subarray** - Fixed size k
- **Longest Turbulent Subarray** - Pattern matching
