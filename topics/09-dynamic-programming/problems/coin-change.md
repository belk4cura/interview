# Coin Change

## 🎯 What Are We Trying To Solve?

Finding the minimum number of coins to make change is like a cashier figuring out the fewest bills and coins to give back - we want the optimal combination.

## 📝 The Problem

Given coins of different denominations and a total amount, find the minimum number of coins needed to make that amount.

```
Example:
coins = [1, 2, 5], amount = 11
Output: 3 (11 = 5 + 5 + 1)
```

## 💻 The Code

```javascript
/**
 * Coin Change - DP Solution
 * Time: O(amount * coins.length), Space: O(amount)
 */
function coinChange(coins, amount) {
    const dp = new Array(amount + 1).fill(Infinity);
    dp[0] = 0;
    
    for (let i = 1; i <= amount; i++) {
        for (const coin of coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    
    return dp[amount] === Infinity ? -1 : dp[amount];
}

// Count ways to make change
function coinChangeWays(coins, amount) {
    const dp = new Array(amount + 1).fill(0);
    dp[0] = 1;
    
    for (const coin of coins) {
        for (let i = coin; i <= amount; i++) {
            dp[i] += dp[i - coin];
        }
    }
    
    return dp[amount];
}
```

## 🎨 Key Insights

- **Unbounded Knapsack**: Can use same coin multiple times
- **DP State**: dp[i] = minimum coins for amount i
- **Build Up**: Start from 0, build to target amount

## ⏱️ Complexity

- Time: O(amount × n) where n = number of coins
- Space: O(amount)
