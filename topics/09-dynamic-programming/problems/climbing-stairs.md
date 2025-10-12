# Climbing Stairs

## 🎯 What Are We Trying To Solve?

Imagine you're climbing a staircase. You can take either 1 step or 2 steps at a time. If the staircase has 3 steps, how many different ways can you reach the top?

This is a classic dynamic programming problem that appears in many real-world scenarios:
- Counting possible paths in a grid
- Calculating Fibonacci-like sequences
- Optimizing decision trees

## 📝 Examples

**Example 1:**
```
Input: n = 2 (Two steps to reach the top)
Output: 2

Explanation: Two ways to climb:
1. Take 1 step + 1 step
2. Take 2 steps at once
```

**Example 2:**
```
Input: n = 3 (Three steps to reach the top)
Output: 3

Explanation: Three ways to climb:
1. Take 1 step + 1 step + 1 step
2. Take 1 step + 2 steps
3. Take 2 steps + 1 step
```

**Example 3:**
```
Input: n = 4
Output: 5

Explanation: Five ways:
1. 1 + 1 + 1 + 1
2. 1 + 1 + 2
3. 1 + 2 + 1
4. 2 + 1 + 1
5. 2 + 2
```

## 🧠 How Do We Think About This?

### Key Insight: Break It Down!

To reach step n, you must come from either:
- Step (n-1) by taking 1 step, OR
- Step (n-2) by taking 2 steps

So: `ways(n) = ways(n-1) + ways(n-2)`

This is the Fibonacci pattern! But let's understand WHY:

```
To reach step 4:
- From step 3 (take 1 step): All ways to reach 3
- From step 2 (take 2 steps): All ways to reach 2
- Total = ways(3) + ways(2)
```

## 💻 The Code

### Approach 1: Recursion (Simple but Slow)
```javascript
/**
 * Recursive solution - Easy to understand but inefficient
 * Time: O(2^n) - Each call branches into 2 more
 * Space: O(n) - Recursion stack depth
 */
function climbStairsRecursive(n) {
    // Base cases
    if (n <= 0) return 0;
    if (n === 1) return 1;  // One way: take 1 step
    if (n === 2) return 2;  // Two ways: 1+1 or 2
    
    // Recursive case: sum of previous two
    return climbStairsRecursive(n - 1) + climbStairsRecursive(n - 2);
}
```

### Approach 2: Dynamic Programming with Array (Good)
```javascript
/**
 * DP with array - Store results to avoid recalculation
 * Time: O(n) - Calculate each step once
 * Space: O(n) - Array to store results
 */
function climbStairsDP(n) {
    // Handle base cases
    if (n <= 0) return 0;
    if (n === 1) return 1;
    
    // Create array to store results
    // dp[i] = number of ways to reach step i
    const dp = new Array(n + 1);
    
    // Base cases
    dp[0] = 1;  // One way to stay at ground (do nothing)
    dp[1] = 1;  // One way to reach step 1
    
    // Build up the solution
    for (let i = 2; i <= n; i++) {
        // Ways to reach step i = 
        // ways to reach (i-1) + ways to reach (i-2)
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    
    return dp[n];
}
```

### Approach 3: Space Optimized (Best!)
```javascript
/**
 * Optimized DP - Only need last two values
 * Time: O(n) - Single pass
 * Space: O(1) - Only two variables
 */
function climbStairs(n) {
    // Handle base cases
    if (n <= 0) return 0;
    if (n === 1) return 1;
    
    // We only need the last two values
    let prev2 = 1;  // Ways to reach step 0
    let prev1 = 1;  // Ways to reach step 1
    
    // Calculate each step
    for (let i = 2; i <= n; i++) {
        // Current = sum of previous two
        const current = prev1 + prev2;
        
        // Shift values for next iteration
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}

// Test all approaches
console.log(climbStairs(2));   // 2
console.log(climbStairs(3));   // 3
console.log(climbStairs(4));   // 5
console.log(climbStairs(5));   // 8
console.log(climbStairs(10));  // 89
```

## 🎨 Step-by-Step Visualization

Let's trace through `climbStairs(4)`:

```
Initial: n = 4, prev2 = 1, prev1 = 1

Step 1 (i = 2):
  - current = prev1 + prev2 = 1 + 1 = 2
  - prev2 = 1, prev1 = 2

Step 2 (i = 3):
  - current = prev1 + prev2 = 2 + 1 = 3
  - prev2 = 2, prev1 = 3

Step 3 (i = 4):
  - current = prev1 + prev2 = 3 + 2 = 5
  - prev2 = 3, prev1 = 5

Return: 5
```

Visual representation of the 5 ways:
```
Way 1: [1] → [1] → [1] → [1]
Way 2: [1] → [1] → [2]
Way 3: [1] → [2] → [1]
Way 4: [2] → [1] → [1]
Way 5: [2] → [2]
```

## ⏱️ How Fast Is This?

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| Recursive | O(2^n) | O(n) | Never in production! Just for understanding |
| DP Array | O(n) | O(n) | When you need all intermediate results |
| Optimized | O(n) | O(1) | Best for this problem |

## 🚫 Common Mistakes

1. **Off-by-one errors with base cases**
   ```javascript
   // Wrong: Forgetting n=0 case
   if (n === 1) return 1;
   if (n === 2) return 2;
   // What if n = 0?
   
   // Right: Handle all base cases
   if (n <= 0) return 0;
   if (n === 1) return 1;
   ```

2. **Not recognizing the Fibonacci pattern**
   ```javascript
   // Some people try complex solutions when it's just Fibonacci!
   // Remember: ways(n) = ways(n-1) + ways(n-2)
   ```

3. **Using recursion without memoization**
   ```javascript
   // This recalculates the same values many times!
   function bad(n) {
       if (n <= 2) return n;
       return bad(n-1) + bad(n-2);  // Exponential time!
   }
   ```

## 🔄 Variations & Extensions

### With Memoization (Caching)
```javascript
function climbStairsMemo(n, memo = {}) {
    // Check cache first
    if (n in memo) return memo[n];
    
    // Base cases
    if (n <= 0) return 0;
    if (n === 1) return 1;
    if (n === 2) return 2;
    
    // Calculate and cache
    memo[n] = climbStairsMemo(n - 1, memo) + climbStairsMemo(n - 2, memo);
    return memo[n];
}
```

### Variable Step Sizes
```javascript
// What if you can take 1, 2, or 3 steps?
function climbStairsVariable(n, steps = [1, 2]) {
    const dp = new Array(n + 1).fill(0);
    dp[0] = 1;
    
    for (let i = 1; i <= n; i++) {
        for (const step of steps) {
            if (i - step >= 0) {
                dp[i] += dp[i - step];
            }
        }
    }
    
    return dp[n];
}

// Can take 1, 2, or 3 steps
console.log(climbStairsVariable(4, [1, 2, 3]));  // 7 ways
```

## 🔗 Related Problems

- **Fibonacci Number**: Direct application of the same pattern
- **House Robber**: Can't rob adjacent houses (similar recurrence)
- **Decode Ways**: Count ways to decode a numeric string
- **Unique Paths**: Count paths in a grid (2D version)
- **Min Cost Climbing Stairs**: Find minimum cost path

## 💡 Key Takeaways

1. **Dynamic Programming Pattern**: Break problem into subproblems
2. **Optimization**: Often only need recent results, not entire history
3. **Fibonacci Connection**: Many counting problems follow this pattern
4. **Bottom-up vs Top-down**: Build from base cases (bottom-up) is often cleaner

This problem is a gateway to understanding dynamic programming. Master this, and you'll recognize the pattern in many other problems!
