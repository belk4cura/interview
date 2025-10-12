# Unique Paths

## 🎯 What Are We Trying To Solve?

Count the number of unique paths from top-left to bottom-right corner of a grid, where you can only move right or down.

## 📝 The Problem

A robot is located at the top-left corner of an `m x n` grid. The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid. How many possible unique paths are there?

```
Example 1:
Input: m = 3, n = 7
Output: 28

Example 2:
Input: m = 3, n = 2
Output: 3
Explanation: From top-left, there are 3 ways:
1. Right -> Down -> Down
2. Down -> Right -> Down  
3. Down -> Down -> Right
```

## 💻 The Code

```javascript
/**
 * Unique Paths - Dynamic Programming
 * Time: O(m*n), Space: O(m*n)
 */
function uniquePaths(m, n) {
    const dp = Array(m).fill().map(() => Array(n).fill(0));
    
    // Initialize first row and column
    for (let i = 0; i < m; i++) {
        dp[i][0] = 1; // Only one way to reach any cell in first column
    }
    for (let j = 0; j < n; j++) {
        dp[0][j] = 1; // Only one way to reach any cell in first row
    }
    
    // Fill the DP table
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }
    
    return dp[m - 1][n - 1];
}

/**
 * Space Optimized - Using 1D array
 * Time: O(m*n), Space: O(n)
 */
function uniquePathsOptimized(m, n) {
    const dp = Array(n).fill(1);
    
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            dp[j] = dp[j] + dp[j - 1];
        }
    }
    
    return dp[n - 1];
}

/**
 * Mathematical Solution - Combinatorics
 * Time: O(min(m,n)), Space: O(1)
 */
function uniquePathsMath(m, n) {
    // Total moves: (m-1) down + (n-1) right = m+n-2
    // Choose (m-1) positions for down moves
    // Answer: C(m+n-2, m-1) = (m+n-2)! / ((m-1)! * (n-1)!)
    
    let result = 1;
    const total = m + n - 2;
    const k = Math.min(m - 1, n - 1);
    
    // Calculate combination
    for (let i = 1; i <= k; i++) {
        result = result * (total - k + i) / i;
    }
    
    return Math.round(result);
}

/**
 * Unique Paths II - With obstacles
 * Time: O(m*n), Space: O(m*n)
 */
function uniquePathsWithObstacles(obstacleGrid) {
    if (!obstacleGrid || obstacleGrid[0][0] === 1) return 0;
    
    const m = obstacleGrid.length;
    const n = obstacleGrid[0].length;
    const dp = Array(m).fill().map(() => Array(n).fill(0));
    
    dp[0][0] = 1;
    
    // Initialize first column
    for (let i = 1; i < m; i++) {
        dp[i][0] = obstacleGrid[i][0] === 1 ? 0 : dp[i - 1][0];
    }
    
    // Initialize first row
    for (let j = 1; j < n; j++) {
        dp[0][j] = obstacleGrid[0][j] === 1 ? 0 : dp[0][j - 1];
    }
    
    // Fill DP table
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            if (obstacleGrid[i][j] === 1) {
                dp[i][j] = 0;
            } else {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
    }
    
    return dp[m - 1][n - 1];
}

/**
 * Minimum Path Sum - Find path with minimum sum
 * Time: O(m*n), Space: O(m*n)
 */
function minPathSum(grid) {
    const m = grid.length;
    const n = grid[0].length;
    const dp = Array(m).fill().map(() => Array(n).fill(0));
    
    dp[0][0] = grid[0][0];
    
    // Initialize first column
    for (let i = 1; i < m; i++) {
        dp[i][0] = dp[i - 1][0] + grid[i][0];
    }
    
    // Initialize first row
    for (let j = 1; j < n; j++) {
        dp[0][j] = dp[0][j - 1] + grid[0][j];
    }
    
    // Fill DP table
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
        }
    }
    
    return dp[m - 1][n - 1];
}

/**
 * Recursive with Memoization
 * Time: O(m*n), Space: O(m*n)
 */
function uniquePathsMemo(m, n) {
    const memo = new Map();
    
    function helper(i, j) {
        // Base case: reached destination
        if (i === m - 1 && j === n - 1) return 1;
        
        // Out of bounds
        if (i >= m || j >= n) return 0;
        
        const key = `${i},${j}`;
        if (memo.has(key)) return memo.get(key);
        
        // Can only move right or down
        const paths = helper(i + 1, j) + helper(i, j + 1);
        memo.set(key, paths);
        
        return paths;
    }
    
    return helper(0, 0);
}

// Test cases
console.log(uniquePaths(3, 7)); // 28
console.log(uniquePaths(3, 2)); // 3
console.log(uniquePaths(7, 3)); // 28
console.log(uniquePathsMath(3, 7)); // 28

const obstacle = [
    [0, 0, 0],
    [0, 1, 0],
    [0, 0, 0]
];
console.log(uniquePathsWithObstacles(obstacle)); // 2

const sumGrid = [
    [1, 3, 1],
    [1, 5, 1],
    [4, 2, 1]
];
console.log(minPathSum(sumGrid)); // 7
```

## 🎨 The Approach

### DP State Definition:
- `dp[i][j]` = Number of unique paths to reach cell (i, j)

### Recurrence Relation:
```
dp[i][j] = dp[i-1][j] + dp[i][j-1]
```
Can reach (i,j) from above or from left

### Visual Walkthrough:
```
Grid: 3 x 3

Start → 1   1   1
        ↓   ↓   ↓
        1   2   3
        ↓   ↓   ↓
        1   3   6 ← End

Explanation:
[0,0] = 1 (start)
[0,1] = 1 (only from left)
[0,2] = 1 (only from left)
[1,0] = 1 (only from above)
[1,1] = 2 (from [0,1] or [1,0])
[1,2] = 3 (from [0,2] or [1,1])
[2,0] = 1 (only from above)
[2,1] = 3 (from [1,1] or [2,0])
[2,2] = 6 (from [1,2] or [2,1])
```

## 🤔 Why This Works

- **Optimal Substructure**: Paths to (i,j) = paths to (i-1,j) + paths to (i,j-1)
- **No backtracking**: Can only move right or down
- **Mathematical insight**: It's a combination problem - choosing positions for moves
- **Grid boundaries**: First row and column have only one path

## ⚠️ Common Pitfalls

1. **Initialization** - First row and column must be 1
2. **Obstacles handling** - Block paths through obstacles
3. **Integer overflow** - Large grids can have huge path counts
4. **Direction confusion** - Only right and down allowed

## 🎯 Variations

### With Obstacles
- Some cells blocked
- Set dp[i][j] = 0 for obstacles

### Minimum Path Sum
- Each cell has a cost
- Find path with minimum total cost

### Maximum Path Sum
- Maximize instead of minimize

### Different Movement Rules
- Allow diagonal moves
- Allow k different move types

## 📊 Complexity Analysis

- **Time**: O(m × n) - Fill entire grid
- **Space**: 
  - Standard: O(m × n)
  - Optimized: O(min(m, n))
  - Math solution: O(1)
- **Mathematical**: O(min(m, n)) time, O(1) space

## 🔄 Related Problems

- **Unique Paths III** - Must visit all non-obstacle cells
- **Minimum Path Sum** - Find path with minimum cost
- **Dungeon Game** - Path with health constraints
- **Cherry Pickup** - Two paths simultaneously
