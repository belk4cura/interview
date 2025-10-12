# Edit Distance

## 🎯 What Are We Trying To Solve?

Find the minimum number of operations (insert, delete, replace) needed to convert one string into another.

## 📝 The Problem

Given two strings `word1` and `word2`, return the minimum number of operations required to convert `word1` to `word2`. You have three operations:
- Insert a character
- Delete a character
- Replace a character

```
Example 1:
Input: word1 = "horse", word2 = "ros"
Output: 3
Explanation: 
horse -> rorse (replace 'h' with 'r')
rorse -> rose (remove 'r')
rose -> ros (remove 'e')

Example 2:
Input: word1 = "intention", word2 = "execution"
Output: 5
Explanation:
intention -> inention (remove 't')
inention -> enention (replace 'i' with 'e')
enention -> exention (replace 'n' with 'x')
exention -> exection (replace 'n' with 'c')
exection -> execution (insert 'u')
```

## 💻 The Code

```javascript
/**
 * Edit Distance - Dynamic Programming
 * Time: O(m*n), Space: O(m*n)
 */
function minDistance(word1, word2) {
    const m = word1.length;
    const n = word2.length;
    
    // Create DP table
    const dp = Array(m + 1).fill().map(() => Array(n + 1).fill(0));
    
    // Base cases: converting to/from empty string
    for (let i = 0; i <= m; i++) {
        dp[i][0] = i; // Delete all characters
    }
    for (let j = 0; j <= n; j++) {
        dp[0][j] = j; // Insert all characters
    }
    
    // Fill DP table
    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (word1[i - 1] === word2[j - 1]) {
                // Characters match, no operation needed
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                // Take minimum of three operations
                dp[i][j] = 1 + Math.min(
                    dp[i - 1][j],     // Delete from word1
                    dp[i][j - 1],     // Insert into word1
                    dp[i - 1][j - 1]  // Replace in word1
                );
            }
        }
    }
    
    return dp[m][n];
}

/**
 * Space Optimized - Using two rows
 * Time: O(m*n), Space: O(min(m,n))
 */
function minDistanceOptimized(word1, word2) {
    // Ensure word1 is shorter for space optimization
    if (word1.length > word2.length) {
        [word1, word2] = [word2, word1];
    }
    
    const m = word1.length;
    const n = word2.length;
    
    let prev = Array(m + 1).fill(0);
    let curr = Array(m + 1).fill(0);
    
    // Initialize first row
    for (let j = 0; j <= m; j++) {
        prev[j] = j;
    }
    
    for (let i = 1; i <= n; i++) {
        curr[0] = i;
        for (let j = 1; j <= m; j++) {
            if (word2[i - 1] === word1[j - 1]) {
                curr[j] = prev[j - 1];
            } else {
                curr[j] = 1 + Math.min(
                    prev[j],      // Delete
                    curr[j - 1],  // Insert
                    prev[j - 1]   // Replace
                );
            }
        }
        [prev, curr] = [curr, prev];
    }
    
    return prev[m];
}

/**
 * Recursive with Memoization
 * Time: O(m*n), Space: O(m*n)
 */
function minDistanceMemo(word1, word2) {
    const memo = new Map();
    
    function helper(i, j) {
        // Base cases
        if (i === 0) return j;
        if (j === 0) return i;
        
        const key = `${i},${j}`;
        if (memo.has(key)) return memo.get(key);
        
        let result;
        if (word1[i - 1] === word2[j - 1]) {
            result = helper(i - 1, j - 1);
        } else {
            result = 1 + Math.min(
                helper(i - 1, j),     // Delete
                helper(i, j - 1),     // Insert
                helper(i - 1, j - 1)  // Replace
            );
        }
        
        memo.set(key, result);
        return result;
    }
    
    return helper(word1.length, word2.length);
}

/**
 * Get actual operations performed
 * Time: O(m*n), Space: O(m*n)
 */
function getEditOperations(word1, word2) {
    const m = word1.length;
    const n = word2.length;
    const dp = Array(m + 1).fill().map(() => Array(n + 1).fill(0));
    
    // Fill DP table
    for (let i = 0; i <= m; i++) dp[i][0] = i;
    for (let j = 0; j <= n; j++) dp[0][j] = j;
    
    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (word1[i - 1] === word2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i - 1][j],
                    dp[i][j - 1],
                    dp[i - 1][j - 1]
                );
            }
        }
    }
    
    // Backtrack to find operations
    const operations = [];
    let i = m, j = n;
    
    while (i > 0 || j > 0) {
        if (i === 0) {
            operations.unshift(`Insert '${word2[j - 1]}' at position ${i}`);
            j--;
        } else if (j === 0) {
            operations.unshift(`Delete '${word1[i - 1]}' at position ${i - 1}`);
            i--;
        } else if (word1[i - 1] === word2[j - 1]) {
            i--;
            j--;
        } else {
            const minOp = Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
            
            if (dp[i - 1][j - 1] === minOp) {
                operations.unshift(`Replace '${word1[i - 1]}' with '${word2[j - 1]}' at position ${i - 1}`);
                i--;
                j--;
            } else if (dp[i - 1][j] === minOp) {
                operations.unshift(`Delete '${word1[i - 1]}' at position ${i - 1}`);
                i--;
            } else {
                operations.unshift(`Insert '${word2[j - 1]}' at position ${i}`);
                j--;
            }
        }
    }
    
    return { distance: dp[m][n], operations };
}

// Test cases
console.log(minDistance("horse", "ros")); // 3
console.log(minDistance("intention", "execution")); // 5
console.log(minDistance("", "abc")); // 3
console.log(minDistance("abc", "")); // 3
console.log(minDistance("abc", "abc")); // 0

const result = getEditOperations("horse", "ros");
console.log(result.distance); // 3
console.log(result.operations);
```

## 🎨 The Approach

### DP State Definition:
- `dp[i][j]` = Minimum operations to convert `word1[0...i-1]` to `word2[0...j-1]`

### Recurrence Relation:
```
If word1[i-1] == word2[j-1]:
    dp[i][j] = dp[i-1][j-1]
Else:
    dp[i][j] = 1 + min(
        dp[i-1][j],     // Delete
        dp[i][j-1],     // Insert
        dp[i-1][j-1]    // Replace
    )
```

### Visual DP Table:
```
    ""  r  o  s
""   0  1  2  3
h    1  1  2  3
o    2  2  1  2
r    3  2  2  2
s    4  3  3  2
e    5  4  4  3
```

## 🤔 Why This Works

- **Optimal Substructure**: Solution for larger strings uses solutions for smaller substrings
- **Three operations map to DP transitions**:
  - Delete: Move up in table (i-1, j)
  - Insert: Move left in table (i, j-1)
  - Replace: Move diagonally (i-1, j-1)
- **Characters match**: No operation needed, inherit from diagonal

## ⚠️ Common Pitfalls

1. **Index confusion** - DP indices vs string indices
2. **Base cases** - Empty string conversions
3. **Operation order** - All three have same cost (1)
4. **Space optimization** - Only need previous row

## 🎯 Key Insights

### Understanding Operations:
- **Delete from word1**: Reduces word1 length
- **Insert into word1**: Matches next char of word2
- **Replace in word1**: Advances both pointers

### Applications:
- Spell checkers
- DNA sequence alignment
- Diff algorithms
- Fuzzy string matching

## 📊 Complexity Analysis

- **Time**: O(m × n) - Fill entire DP table
- **Space**: 
  - Standard: O(m × n)
  - Optimized: O(min(m, n))
- **Optimal**: Can't do better for exact distance

## 🔄 Related Problems

- **Longest Common Subsequence** - Similar DP structure
- **Minimum ASCII Delete Sum** - Weighted edit distance
- **One Edit Distance** - Check if exactly one edit away
- **Distinct Subsequences** - Count transformations
