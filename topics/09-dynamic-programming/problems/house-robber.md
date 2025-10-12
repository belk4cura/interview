# House Robber

## 🎯 What Are We Trying To Solve?

Maximize the amount of money you can rob from houses on a street, where you cannot rob two adjacent houses due to security systems.

## 📝 The Problem

You are a professional robber planning to rob houses along a street. Each house has a certain amount of money. Adjacent houses have security systems connected - if two adjacent houses are broken into on the same night, the police will be alerted.

Given an array representing money in each house, return the maximum amount you can rob without alerting the police.

```
Example 1:
Input: nums = [1,2,3,1]
Output: 4
Explanation: Rob house 1 (money = 1) and house 3 (money = 3)

Example 2:
Input: nums = [2,7,9,3,1]
Output: 12
Explanation: Rob house 1 (money = 2), house 3 (money = 9) and house 5 (money = 1)

Example 3:
Input: nums = [2,1,1,2]
Output: 4
Explanation: Rob house 1 (money = 2) and house 4 (money = 2)
```

## 💻 The Code

```javascript
/**
 * House Robber - Dynamic Programming
 * Time: O(n), Space: O(n)
 */
function rob(nums) {
    if (nums.length === 0) return 0;
    if (nums.length === 1) return nums[0];
    
    const dp = new Array(nums.length);
    dp[0] = nums[0];
    dp[1] = Math.max(nums[0], nums[1]);
    
    for (let i = 2; i < nums.length; i++) {
        // Either rob current house + i-2, or don't rob and take i-1
        dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
    }
    
    return dp[nums.length - 1];
}

/**
 * Space Optimized - Only track last two values
 * Time: O(n), Space: O(1)
 */
function robOptimized(nums) {
    if (nums.length === 0) return 0;
    if (nums.length === 1) return nums[0];
    
    let prev2 = nums[0];
    let prev1 = Math.max(nums[0], nums[1]);
    
    for (let i = 2; i < nums.length; i++) {
        const current = Math.max(prev2 + nums[i], prev1);
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}

/**
 * House Robber II - Houses in circle
 * Time: O(n), Space: O(1)
 */
function robCircular(nums) {
    if (nums.length === 0) return 0;
    if (nums.length === 1) return nums[0];
    if (nums.length === 2) return Math.max(nums[0], nums[1]);
    
    // Can't rob both first and last house
    // Case 1: Rob houses 0 to n-2
    const robFirst = robRange(nums, 0, nums.length - 2);
    // Case 2: Rob houses 1 to n-1
    const robLast = robRange(nums, 1, nums.length - 1);
    
    return Math.max(robFirst, robLast);
}

function robRange(nums, start, end) {
    let prev2 = 0;
    let prev1 = 0;
    
    for (let i = start; i <= end; i++) {
        const current = Math.max(prev2 + nums[i], prev1);
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}

/**
 * House Robber III - Binary Tree
 * Time: O(n), Space: O(n) for recursion
 */
function robTree(root) {
    // Returns [maxWithRoot, maxWithoutRoot]
    function dfs(node) {
        if (!node) return [0, 0];
        
        const [leftWith, leftWithout] = dfs(node.left);
        const [rightWith, rightWithout] = dfs(node.right);
        
        // If we rob this node, we can't rob children
        const robThis = node.val + leftWithout + rightWithout;
        
        // If we don't rob this node, we take max from children
        const skipThis = Math.max(leftWith, leftWithout) + 
                        Math.max(rightWith, rightWithout);
        
        return [robThis, skipThis];
    }
    
    const [withRoot, withoutRoot] = dfs(root);
    return Math.max(withRoot, withoutRoot);
}

/**
 * Recursive with Memoization
 * Time: O(n), Space: O(n)
 */
function robMemo(nums) {
    const memo = new Map();
    
    function helper(i) {
        if (i >= nums.length) return 0;
        if (memo.has(i)) return memo.get(i);
        
        // Rob current house or skip it
        const result = Math.max(
            nums[i] + helper(i + 2),  // Rob current
            helper(i + 1)              // Skip current
        );
        
        memo.set(i, result);
        return result;
    }
    
    return helper(0);
}

// Test cases
console.log(rob([1,2,3,1])); // 4
console.log(rob([2,7,9,3,1])); // 12
console.log(rob([2,1,1,2])); // 4
console.log(robOptimized([5,1,3,9,4])); // 14
console.log(robCircular([2,3,2])); // 3
console.log(robCircular([1,2,3,1])); // 4
```

## 🎨 The Approach

### DP State Definition:
- `dp[i]` = Maximum money that can be robbed from houses 0 to i

### Recurrence Relation:
```
dp[i] = max(
    dp[i-2] + nums[i],  // Rob house i
    dp[i-1]             // Skip house i
)
```

### Visual Walkthrough:
```
Houses: [2, 7, 9, 3, 1]
         0  1  2  3  4

dp[0] = 2 (rob house 0)
dp[1] = max(2, 7) = 7 (rob house 1, skip 0)
dp[2] = max(2+9, 7) = 11 (rob houses 0,2)
dp[3] = max(7+3, 11) = 11 (skip house 3)
dp[4] = max(11+1, 11) = 12 (rob houses 0,2,4)

Result: 12
```

## 🤔 Why This Works

- **Optimal Substructure**: Best solution for n houses uses best solution for n-1 or n-2 houses
- **No adjacent constraint**: Can only use i-2 if robbing house i
- **Greedy doesn't work**: Can't just pick largest values
- **Overlapping subproblems**: Same subproblems solved multiple times

## ⚠️ Common Pitfalls

1. **Edge cases** - Empty array, single house
2. **Initial values** - dp[0] and dp[1] need special handling
3. **Space optimization** - Only need last two values
4. **Circular variant** - First and last house connected

## 🎯 Variations

### House Robber II (Circular)
- Houses arranged in circle
- Can't rob both first and last house
- Solution: Max of two linear problems

### House Robber III (Tree)
- Houses in binary tree structure
- Can't rob directly connected nodes
- Solution: DFS with two states per node

## 📊 Complexity Analysis

- **Time**: O(n) - Single pass through array
- **Space**: 
  - DP array: O(n)
  - Optimized: O(1)
- **Optimal**: Can't do better than O(n)

## 🔄 Related Problems

- **Maximum Non-Adjacent Sum** - Same concept
- **Delete and Earn** - Transform to house robber
- **Paint House** - Similar DP pattern
- **Fibonacci** - Similar recurrence relation
