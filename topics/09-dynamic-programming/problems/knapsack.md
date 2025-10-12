# 0/1 Knapsack Problem

## 🎯 What Are We Trying To Solve?

Maximize the value of items that can fit in a knapsack with limited capacity, where each item can be taken at most once.

## 📝 The Problem

Given weights and values of n items, put these items in a knapsack of capacity W to get the maximum total value. You cannot break an item, either pick it or don't pick it (0/1 property).

```
Example 1:
Input: 
weights = [1, 3, 4, 5]
values = [1, 4, 5, 7]
capacity = 7
Output: 9
Explanation: Take items with weights 3 and 4 (values 4 and 5)

Example 2:
Input:
weights = [10, 20, 30]
values = [60, 100, 120]
capacity = 50
Output: 220
Explanation: Take items with weights 20 and 30
```

## 💻 The Code

```javascript
/**
 * 0/1 Knapsack - Dynamic Programming
 * Time: O(n*W), Space: O(n*W)
 */
function knapsack(weights, values, capacity) {
    const n = weights.length;
    // dp[i][w] = max value with first i items and capacity w
    const dp = Array(n + 1).fill().map(() => Array(capacity + 1).fill(0));
    
    for (let i = 1; i <= n; i++) {
        for (let w = 0; w <= capacity; w++) {
            // Don't take current item
            dp[i][w] = dp[i - 1][w];
            
            // Take current item if it fits
            if (weights[i - 1] <= w) {
                const valueWithItem = values[i - 1] + dp[i - 1][w - weights[i - 1]];
                dp[i][w] = Math.max(dp[i][w], valueWithItem);
            }
        }
    }
    
    return dp[n][capacity];
}

/**
 * Space Optimized - Using 1D array
 * Time: O(n*W), Space: O(W)
 */
function knapsackOptimized(weights, values, capacity) {
    const n = weights.length;
    const dp = Array(capacity + 1).fill(0);
    
    for (let i = 0; i < n; i++) {
        // Traverse backwards to avoid using updated values
        for (let w = capacity; w >= weights[i]; w--) {
            dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }
    
    return dp[capacity];
}

/**
 * Get actual items selected
 * Time: O(n*W), Space: O(n*W)
 */
function knapsackWithItems(weights, values, capacity) {
    const n = weights.length;
    const dp = Array(n + 1).fill().map(() => Array(capacity + 1).fill(0));
    
    // Fill DP table
    for (let i = 1; i <= n; i++) {
        for (let w = 0; w <= capacity; w++) {
            dp[i][w] = dp[i - 1][w];
            if (weights[i - 1] <= w) {
                dp[i][w] = Math.max(
                    dp[i][w],
                    values[i - 1] + dp[i - 1][w - weights[i - 1]]
                );
            }
        }
    }
    
    // Backtrack to find items
    const items = [];
    let w = capacity;
    for (let i = n; i > 0 && w > 0; i--) {
        if (dp[i][w] !== dp[i - 1][w]) {
            items.push({
                index: i - 1,
                weight: weights[i - 1],
                value: values[i - 1]
            });
            w -= weights[i - 1];
        }
    }
    
    return {
        maxValue: dp[n][capacity],
        items: items.reverse()
    };
}

/**
 * Recursive with Memoization
 * Time: O(n*W), Space: O(n*W)
 */
function knapsackMemo(weights, values, capacity) {
    const n = weights.length;
    const memo = new Map();
    
    function helper(i, remainingCapacity) {
        // Base case
        if (i === n || remainingCapacity === 0) return 0;
        
        const key = `${i},${remainingCapacity}`;
        if (memo.has(key)) return memo.get(key);
        
        // Don't take current item
        let result = helper(i + 1, remainingCapacity);
        
        // Take current item if it fits
        if (weights[i] <= remainingCapacity) {
            const valueWithItem = values[i] + 
                helper(i + 1, remainingCapacity - weights[i]);
            result = Math.max(result, valueWithItem);
        }
        
        memo.set(key, result);
        return result;
    }
    
    return helper(0, capacity);
}

/**
 * Unbounded Knapsack - Can use items multiple times
 * Time: O(n*W), Space: O(W)
 */
function unboundedKnapsack(weights, values, capacity) {
    const dp = Array(capacity + 1).fill(0);
    
    for (let w = 0; w <= capacity; w++) {
        for (let i = 0; i < weights.length; i++) {
            if (weights[i] <= w) {
                dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
            }
        }
    }
    
    return dp[capacity];
}

/**
 * Fractional Knapsack - Can take fractions (Greedy)
 * Time: O(n log n), Space: O(n)
 */
function fractionalKnapsack(weights, values, capacity) {
    const n = weights.length;
    const items = [];
    
    // Calculate value/weight ratio
    for (let i = 0; i < n; i++) {
        items.push({
            index: i,
            weight: weights[i],
            value: values[i],
            ratio: values[i] / weights[i]
        });
    }
    
    // Sort by ratio in descending order
    items.sort((a, b) => b.ratio - a.ratio);
    
    let totalValue = 0;
    let remainingCapacity = capacity;
    const selected = [];
    
    for (const item of items) {
        if (remainingCapacity === 0) break;
        
        if (item.weight <= remainingCapacity) {
            // Take whole item
            totalValue += item.value;
            remainingCapacity -= item.weight;
            selected.push({ ...item, fraction: 1 });
        } else {
            // Take fraction
            const fraction = remainingCapacity / item.weight;
            totalValue += item.value * fraction;
            selected.push({ ...item, fraction });
            remainingCapacity = 0;
        }
    }
    
    return { totalValue, selected };
}

// Test cases
console.log(knapsack([1, 3, 4, 5], [1, 4, 5, 7], 7)); // 9
console.log(knapsack([10, 20, 30], [60, 100, 120], 50)); // 220
console.log(knapsackOptimized([2, 1, 3, 2], [12, 10, 20, 15], 5)); // 37

const result = knapsackWithItems([1, 3, 4, 5], [1, 4, 5, 7], 7);
console.log(result); // {maxValue: 9, items: [...]}

console.log(unboundedKnapsack([1, 3, 4, 5], [10, 40, 50, 70], 8)); // 110
console.log(fractionalKnapsack([10, 20, 30], [60, 100, 120], 50));
```

## 🎨 The Approach

### DP State Definition:
- `dp[i][w]` = Maximum value using first i items with capacity w

### Recurrence Relation:
```
dp[i][w] = max(
    dp[i-1][w],                              // Don't take item i
    values[i-1] + dp[i-1][w-weights[i-1]]   // Take item i
)
```

### Visual DP Table:
```
capacity →  0  1  2  3  4  5  6  7
items ↓     
[]          0  0  0  0  0  0  0  0
[1,1]       0  1  1  1  1  1  1  1
[3,4]       0  1  1  4  5  5  5  5
[4,5]       0  1  1  4  5  6  6  9
[5,7]       0  1  1  4  5  7  8  9

Result: dp[4][7] = 9
```

## 🤔 Why This Works

- **Optimal Substructure**: Optimal solution uses optimal solutions of subproblems
- **Overlapping Subproblems**: Same subproblems solved multiple times
- **Decision at each item**: Include or exclude
- **Capacity constraint**: Must respect weight limit

## ⚠️ Common Pitfalls

1. **Array indexing** - Item i-1 corresponds to dp[i]
2. **Backwards iteration** - Required for space optimization
3. **Integer weights** - Fractional weights need different approach
4. **Value vs weight** - Don't confuse the two arrays

## 🎯 Variations

### Unbounded Knapsack
- Can use items multiple times
- Different DP recurrence

### Fractional Knapsack
- Can take fractions of items
- Greedy approach works

### Multiple Knapsacks
- Distribute items among multiple knapsacks

### Subset Sum
- Special case where values equal weights

## 📊 Complexity Analysis

- **Time**: O(n × W) - Fill entire DP table
- **Space**: 
  - Standard: O(n × W)
  - Optimized: O(W)
- **Pseudo-polynomial**: Depends on capacity value

## 🔄 Related Problems

- **Subset Sum** - Can we make target sum?
- **Partition Equal Subset** - Divide into equal sum subsets
- **Coin Change** - Unbounded knapsack variant
- **Target Sum** - Assign +/- to make target
