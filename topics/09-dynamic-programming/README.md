# 💎 Dynamic Programming

## Overview
Dynamic Programming (DP) is an optimization technique that solves complex problems by breaking them into overlapping subproblems and storing their solutions. It transforms exponential-time recursive solutions into polynomial-time algorithms.

## Key Concepts

### DP Characteristics
1. **Overlapping Subproblems**: Same subproblems solved multiple times
2. **Optimal Substructure**: Optimal solution contains optimal solutions to subproblems
3. **Memoization**: Top-down approach with caching
4. **Tabulation**: Bottom-up approach with table building

### DP Approaches

| Approach | Direction | Storage | When to Use |
|----------|-----------|---------|-------------|
| Top-Down (Memoization) | Recursive | Hash/Array | Natural recursion, sparse subproblems |
| Bottom-Up (Tabulation) | Iterative | Array/Table | Dense subproblems, space optimization |

## Common Patterns

### 1. Linear DP
```javascript
// Classic: Fibonacci
function fib(n) {
    const dp = [0, 1];
    for (let i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }
    return dp[n];
}

// Space optimized
function fibOptimized(n) {
    if (n <= 1) return n;
    let prev2 = 0, prev1 = 1;
    for (let i = 2; i <= n; i++) {
        const curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

### 2. Grid DP
```javascript
// Unique paths in grid
function uniquePaths(m, n) {
    const dp = Array(m).fill().map(() => Array(n).fill(1));
    
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
        }
    }
    
    return dp[m-1][n-1];
}
```

### 3. Sequence DP
```javascript
// Longest Common Subsequence
function LCS(text1, text2) {
    const m = text1.length, n = text2.length;
    const dp = Array(m + 1).fill().map(() => Array(n + 1).fill(0));
    
    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (text1[i-1] === text2[j-1]) {
                dp[i][j] = dp[i-1][j-1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
            }
        }
    }
    
    return dp[m][n];
}
```

### 4. Interval DP
```javascript
// Matrix Chain Multiplication
function matrixChainOrder(dims) {
    const n = dims.length - 1;
    const dp = Array(n).fill().map(() => Array(n).fill(0));
    
    for (let len = 2; len <= n; len++) {
        for (let i = 0; i < n - len + 1; i++) {
            const j = i + len - 1;
            dp[i][j] = Infinity;
            
            for (let k = i; k < j; k++) {
                const cost = dp[i][k] + dp[k+1][j] + 
                           dims[i] * dims[k+1] * dims[j+1];
                dp[i][j] = Math.min(dp[i][j], cost);
            }
        }
    }
    
    return dp[0][n-1];
}
```

## Problems in This Section

### Fundamental DP
- [Climbing Stairs](problems/climbing-stairs.md) - Classic DP introduction ⭐
- [House Robber](problems/house-robber.md) - Max non-adjacent sum ⭐
- [Maximum Subarray](problems/maximum-subarray.md) - Kadane's algorithm ⭐

### Grid & Path DP
- [Unique Paths](problems/unique-paths.md) - Grid traversal counting ⭐

### String & Sequence DP
- [Edit Distance](problems/edit-distance.md) - Levenshtein distance ⭐
- [Longest Increasing Subsequence](problems/longest-increasing-subsequence.md) - Classic sequence DP ⭐
- [Word Break](problems/word-break.md) - String segmentation ⭐

### Knapsack Problems
- [0/1 Knapsack](problems/knapsack.md) - Fundamental knapsack problem ⭐
- [Coin Change](problems/coin-change.md) - Unbounded knapsack variant ⭐

## Classic DP Problems

### 0/1 Knapsack
```javascript
function knapsack(weights, values, capacity) {
    const n = weights.length;
    const dp = Array(n + 1).fill().map(() => 
        Array(capacity + 1).fill(0)
    );
    
    for (let i = 1; i <= n; i++) {
        for (let w = 0; w <= capacity; w++) {
            // Don't take item i-1
            dp[i][w] = dp[i-1][w];
            
            // Take item i-1 if possible
            if (weights[i-1] <= w) {
                dp[i][w] = Math.max(
                    dp[i][w],
                    dp[i-1][w - weights[i-1]] + values[i-1]
                );
            }
        }
    }
    
    return dp[n][capacity];
}
```

### Edit Distance
```javascript
function editDistance(word1, word2) {
    const m = word1.length, n = word2.length;
    const dp = Array(m + 1).fill().map(() => Array(n + 1).fill(0));
    
    // Initialize base cases
    for (let i = 0; i <= m; i++) dp[i][0] = i;
    for (let j = 0; j <= n; j++) dp[0][j] = j;
    
    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (word1[i-1] === word2[j-1]) {
                dp[i][j] = dp[i-1][j-1];
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i-1][j],    // Delete
                    dp[i][j-1],    // Insert
                    dp[i-1][j-1]   // Replace
                );
            }
        }
    }
    
    return dp[m][n];
}
```

### House Robber
```javascript
function rob(nums) {
    if (nums.length === 0) return 0;
    if (nums.length === 1) return nums[0];
    
    let prev2 = nums[0];
    let prev1 = Math.max(nums[0], nums[1]);
    
    for (let i = 2; i < nums.length; i++) {
        const curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    
    return prev1;
}
```

## State Definition Tips

1. **What decisions do I make?** → State dimensions
2. **What info do I need?** → State parameters
3. **What am I optimizing?** → DP value
4. **How do states relate?** → Transition function

### Common State Patterns
- `dp[i]` = Solution for first i elements
- `dp[i][j]` = Solution for range [i, j]
- `dp[i][j]` = Solution for first i of A and first j of B
- `dp[i][sum]` = Can we make sum with first i elements?
- `dp[mask]` = Solution for subset represented by bitmask

## Space Optimization

### Rolling Array
```javascript
// 2D to 1D optimization
function optimizedDP(n) {
    // Instead of dp[i][j], use dp[j] and prev[j]
    let prev = new Array(n).fill(0);
    let curr = new Array(n).fill(0);
    
    for (let i = 0; i < n; i++) {
        for (let j = 0; j < n; j++) {
            curr[j] = /* calculate based on prev */;
        }
        [prev, curr] = [curr, prev];
    }
    
    return prev[n-1];
}
```

## When to Use DP

✅ **Perfect for:**
- Optimization problems (min/max)
- Counting problems
- Decision problems (yes/no)
- Problems with overlapping subproblems
- Problems with optimal substructure

❌ **Not suitable when:**
- No overlapping subproblems
- Greedy solution exists
- Problem size too large for memory
- No clear state definition

## Interview Strategy

1. **Recognize DP** - Keywords: optimal, maximum, minimum, count ways
2. **Start with recursion** - Define recursive relation
3. **Add memoization** - Cache results
4. **Convert to tabulation** - If needed
5. **Optimize space** - Rolling arrays, dimension reduction

## Common DP Types

- **Linear DP**: Fibonacci, House Robber
- **Grid DP**: Unique Paths, Min Path Sum  
- **String DP**: Edit Distance, LCS
- **Tree DP**: Diameter, Path Sum
- **Interval DP**: Burst Balloons, Matrix Chain
- **Digit DP**: Count numbers with property
- **Bitmask DP**: Traveling Salesman
- **Probability DP**: Expected value problems
