# Ice Cream Parlor

## 🎯 What Are We Trying To Solve?

You're at an ice cream parlor with exactly `m` dollars to spend. You must buy exactly two different flavors that together cost all your money. It's like finding the perfect pair that uses your entire budget - no change, no leftover money!

## 📝 The Problem

Given an array of ice cream prices and a budget `m`, find two different ice creams whose prices sum exactly to `m`. Return the 1-based indices of the two ice creams.

```
Example:
Budget: $4
Prices: [1, 4, 5, 3, 2]

Ice creams at index 0 ($1) and 3 ($3) = $4 ✓
Return: [1, 4] (1-based indices)
```

## 📊 Examples

**Example 1:**
```
Input: m = 4, prices = [1, 4, 5, 3, 2]
Output: [1, 4]

Explanation:
prices[0] + prices[3] = 1 + 3 = 4
1-based indices: [1, 4]
```

**Example 2:**
```
Input: m = 4, prices = [2, 2, 4, 3]
Output: [1, 2]

Explanation:
prices[0] + prices[1] = 2 + 2 = 4
Both cost $2, buy both!
```

## 💻 The Code

```javascript
/**
 * Find two ice creams that cost exactly m dollars
 * @param {number} m - Total money to spend
 * @param {number[]} prices - Array of ice cream prices
 * @returns {number[]} - 1-based indices of two ice creams
 */
function iceCreamParlor(m, prices) {
    // Map to store price -> indices
    const priceMap = new Map();
    
    // Build the map with all indices for each price
    for (let i = 0; i < prices.length; i++) {
        if (!priceMap.has(prices[i])) {
            priceMap.set(prices[i], []);
        }
        priceMap.get(prices[i]).push(i);
    }
    
    // Find the pair
    for (let i = 0; i < prices.length; i++) {
        const currentPrice = prices[i];
        const neededPrice = m - currentPrice;
        
        if (priceMap.has(neededPrice)) {
            const indices = priceMap.get(neededPrice);
            
            // If same price needed twice, check we have at least 2
            if (currentPrice === neededPrice) {
                if (indices.length > 1) {
                    // Return first two indices (1-based)
                    return [i + 1, indices[1] + 1];
                }
            } else {
                // Different prices, return both indices
                return [i + 1, indices[0] + 1];
            }
        }
    }
    
    return [];  // No solution found
}

// Alternative: One-pass solution with hash map
function iceCreamParlorOnePass(m, prices) {
    const seen = new Map();
    
    for (let i = 0; i < prices.length; i++) {
        const complement = m - prices[i];
        
        if (seen.has(complement)) {
            // Found the pair! Return 1-based indices
            return [seen.get(complement) + 1, i + 1];
        }
        
        // Store current price with its index
        seen.set(prices[i], i);
    }
    
    return [];
}

// Alternative: Brute force O(n²) solution
function iceCreamParlorBrute(m, prices) {
    for (let i = 0; i < prices.length - 1; i++) {
        for (let j = i + 1; j < prices.length; j++) {
            if (prices[i] + prices[j] === m) {
                return [i + 1, j + 1];  // 1-based indices
            }
        }
    }
    return [];
}

// Test cases
console.log(iceCreamParlor(4, [1, 4, 5, 3, 2]));  // [1, 4]
console.log(iceCreamParlor(4, [2, 2, 4, 3]));     // [1, 2]
console.log(iceCreamParlor(5, [1, 2, 3, 4]));     // [2, 4]
console.log(iceCreamParlor(10, [5, 5]));          // [1, 2]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `iceCreamParlor(4, [1, 4, 5, 3, 2])`:

```
Budget: 4
Prices: [1, 4, 5, 3, 2]

Step 1: Build price map
priceMap = {
    1: [0],
    4: [1],
    5: [2],
    3: [3],
    2: [4]
}

Step 2: Find the pair
i = 0, price = 1
- Need: 4 - 1 = 3
- Check map: 3 exists at index [3]
- Return [0+1, 3+1] = [1, 4] ✓

The ice creams at positions 1 and 4 (1-based)
cost $1 and $3, totaling exactly $4!
```

## 📊 Visual Understanding

```
One-Pass Algorithm:

Budget: 4
Prices: [2, 2, 4, 3]

Pass:
i=0: price=2, need=2
     seen={} → not found
     seen={2:0}

i=1: price=2, need=2
     seen={2:0} → found at index 0!
     Return [1, 2]

Visual:
[2, 2, 4, 3]
 ↑  ↑
 $2 $2 = $4 ✓
```

## ⏱️ Complexity Analysis

**Hash Map Solution:**
- Time: O(n) - Single pass through array
- Space: O(n) - Hash map storage

**Brute Force:**
- Time: O(n²) - Nested loops
- Space: O(1) - No extra storage

The hash map trades space for time efficiency!

## 🚫 Common Mistakes

1. **Returning 0-Based Instead of 1-Based Indices**
   ```javascript
   // Wrong: Returning 0-based indices
   return [i, j];
   
   // Right: Convert to 1-based
   return [i + 1, j + 1];
   ```

2. **Not Handling Same Price Twice**
   ```javascript
   // Wrong: Using same index twice
   if (prices[i] === m/2) {
       return [i, i];  // Can't buy same ice cream twice!
   }
   
   // Right: Check for another ice cream with same price
   if (indices.length > 1) {
       return [i + 1, indices[1] + 1];
   }
   ```

3. **Not Considering Order**
   ```javascript
   // Make sure to return indices in ascending order
   return [Math.min(i, j) + 1, Math.max(i, j) + 1];
   ```

## 🔄 Variations

```javascript
// Find all pairs that sum to budget
function allIceCreamPairs(m, prices) {
    const pairs = [];
    const seen = new Map();
    
    for (let i = 0; i < prices.length; i++) {
        const complement = m - prices[i];
        
        if (seen.has(complement)) {
            for (const j of seen.get(complement)) {
                pairs.push([j + 1, i + 1]);
            }
        }
        
        if (!seen.has(prices[i])) {
            seen.set(prices[i], []);
        }
        seen.get(prices[i]).push(i);
    }
    
    return pairs;
}

// Find pair closest to budget without going over
function bestIceCreamDeal(m, prices) {
    let bestPair = [];
    let bestSum = 0;
    
    for (let i = 0; i < prices.length - 1; i++) {
        for (let j = i + 1; j < prices.length; j++) {
            const sum = prices[i] + prices[j];
            if (sum <= m && sum > bestSum) {
                bestSum = sum;
                bestPair = [i + 1, j + 1];
            }
        }
    }
    
    return bestPair;
}

// Can buy k ice creams with budget m
function kIceCreams(m, prices, k) {
    // This becomes a k-sum problem
    // Use recursion or dynamic programming
}
```

## 🔗 Related Problems

- **Two Sum** - The classic version of this problem
- **3Sum** - Find three numbers that sum to target
- **Subarray Sum Equals K** - Continuous subarray version
- **Coin Change** - Making exact change with coins

## 💡 Key Takeaways

1. **Hash Map Pattern**: Store complement values for O(1) lookup
2. **Handle Duplicates**: Same price might appear multiple times
3. **Index Conversion**: Problems often want 1-based indices
4. **One-Pass Optimization**: Build map while searching

This is essentially the Two Sum problem with a fun ice cream twist - a great example of how classic algorithms appear in real-world scenarios!
