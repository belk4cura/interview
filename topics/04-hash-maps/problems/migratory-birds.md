# Migratory Birds

## 🎯 What Are We Trying To Solve?

You're tracking bird sightings where each bird has a type ID (1-5). You need to find which type of bird is most frequently sighted. If there's a tie, choose the bird with the smallest type ID. It's like finding the most popular bird in your birdwatching data!

## 📝 The Problem

Given an array of bird sightings (where each number represents a bird type from 1 to 5), find the type that occurs most frequently. If multiple types have the maximum frequency, return the smallest type number.

```
Example:
Sightings: [1, 4, 4, 4, 5, 3]

Count:
Type 1: 1 sighting
Type 3: 1 sighting  
Type 4: 3 sightings ← Most frequent
Type 5: 1 sighting

Return: 4
```

## 📊 Examples

**Example 1:**
```
Input: [1, 4, 4, 4, 5, 3]
Output: 4

Type counts: {1:1, 3:1, 4:3, 5:1}
Most frequent: Type 4 with 3 sightings
```

**Example 2:**
```
Input: [1, 2, 3, 4, 5, 4, 3, 2, 1, 3, 4]
Output: 3

Type counts: {1:2, 2:2, 3:3, 4:3, 5:1}
Tie between 3 and 4 (both have 3)
Choose smaller: 3
```

## 💻 The Code

```javascript
/**
 * Find the most common bird type
 * @param {number[]} birds - Array of bird type IDs (1-5)
 * @returns {number} - Most common type (smallest if tie)
 */
function migratoryBirds(birds) {
    // Count frequency of each bird type
    const counts = new Map();
    
    for (const bird of birds) {
        counts.set(bird, (counts.get(bird) || 0) + 1);
    }
    
    // Find the type with maximum count
    let maxCount = 0;
    let mostCommonType = 1;
    
    // Check types 1-5 in order (ensures smallest type wins ties)
    for (let type = 1; type <= 5; type++) {
        const count = counts.get(type) || 0;
        if (count > maxCount) {
            maxCount = count;
            mostCommonType = type;
        }
    }
    
    return mostCommonType;
}

// Alternative: Using array for fixed types (1-5)
function migratoryBirdsArray(birds) {
    // Index 0-4 represent types 1-5
    const counts = [0, 0, 0, 0, 0];
    
    // Count each bird type
    for (const bird of birds) {
        counts[bird - 1]++;
    }
    
    // Find maximum count and its type
    let maxCount = 0;
    let mostCommonType = 1;
    
    for (let i = 0; i < 5; i++) {
        if (counts[i] > maxCount) {
            maxCount = counts[i];
            mostCommonType = i + 1;
        }
    }
    
    return mostCommonType;
}

// Alternative: General solution for any number of types
function migratoryBirdsGeneral(birds) {
    const counts = {};
    
    // Count frequencies
    for (const bird of birds) {
        counts[bird] = (counts[bird] || 0) + 1;
    }
    
    // Find max frequency
    const maxCount = Math.max(...Object.values(counts));
    
    // Find all types with max frequency
    const mostCommon = Object.keys(counts)
        .filter(type => counts[type] === maxCount)
        .map(Number);
    
    // Return smallest type
    return Math.min(...mostCommon);
}

// Alternative: Using reduce
function migratoryBirdsReduce(birds) {
    // Count frequencies
    const counts = birds.reduce((acc, bird) => {
        acc[bird] = (acc[bird] || 0) + 1;
        return acc;
    }, {});
    
    // Find type with max count (smallest if tie)
    return Object.entries(counts)
        .sort((a, b) => {
            // Sort by count descending, then by type ascending
            if (b[1] !== a[1]) return b[1] - a[1];
            return a[0] - b[0];
        })[0][0];
}

// Test cases
console.log(migratoryBirds([1, 4, 4, 4, 5, 3]));        // 4
console.log(migratoryBirds([1, 2, 3, 4, 5, 4, 3, 2, 1, 3, 4])); // 3
console.log(migratoryBirds([1, 1, 2, 2, 3]));           // 1 (tie, choose smaller)
console.log(migratoryBirds([5, 5, 5, 5, 5]));           // 5
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `migratoryBirds([1, 2, 3, 4, 5, 4, 3, 2, 1, 3, 4])`:

```
Input: [1, 2, 3, 4, 5, 4, 3, 2, 1, 3, 4]

Step 1: Count each bird type
Processing array:
- 1 appears → counts = {1: 1}
- 2 appears → counts = {1: 1, 2: 1}
- 3 appears → counts = {1: 1, 2: 1, 3: 1}
- 4 appears → counts = {1: 1, 2: 1, 3: 1, 4: 1}
- 5 appears → counts = {1: 1, 2: 1, 3: 1, 4: 1, 5: 1}
- 4 appears → counts = {1: 1, 2: 1, 3: 1, 4: 2, 5: 1}
- 3 appears → counts = {1: 1, 2: 1, 3: 2, 4: 2, 5: 1}
- 2 appears → counts = {1: 1, 2: 2, 3: 2, 4: 2, 5: 1}
- 1 appears → counts = {1: 2, 2: 2, 3: 2, 4: 2, 5: 1}
- 3 appears → counts = {1: 2, 2: 2, 3: 3, 4: 2, 5: 1}
- 4 appears → counts = {1: 2, 2: 2, 3: 3, 4: 3, 5: 1}

Step 2: Find maximum count
Type 1: 2 sightings
Type 2: 2 sightings
Type 3: 3 sightings ← Maximum
Type 4: 3 sightings ← Also maximum
Type 5: 1 sighting

Step 3: Handle tie
Types 3 and 4 both have 3 sightings
Choose smaller: 3

Return: 3
```

## 📊 Visual Understanding

```
Bird Sightings Frequency Chart:

Type 1: ██        (2)
Type 2: ██        (2)
Type 3: ███       (3) ← Winner (tie with 4, but smaller)
Type 4: ███       (3)
Type 5: █         (1)

Processing Order Matters:
Check types 1→2→3→4→5
First maximum found: Type 3
Later tie with Type 4 doesn't override
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(n)
  - Single pass to count frequencies
  - Constant time to check 5 types
  
- **Space Complexity:** O(1)
  - Only 5 possible bird types
  - Fixed-size storage regardless of input size

## 🚫 Common Mistakes

1. **Not Handling Ties Correctly**
   ```javascript
   // Wrong: Returns last max found
   for (const [type, count] of counts) {
       if (count >= maxCount) {  // >= means last wins
           mostCommon = type;
       }
   }
   
   // Right: Check in order, use > not >=
   for (let type = 1; type <= 5; type++) {
       if (counts[type] > maxCount) {  // > means first wins ties
   ```

2. **Off-by-One with Array Indexing**
   ```javascript
   // Wrong: Forgetting to adjust for 0-based indexing
   counts[bird]++;  // If bird=1, this accesses index 1, not 0
   
   // Right: Adjust for 0-based arrays
   counts[bird - 1]++;
   ```

3. **Not Initializing Counts**
   ```javascript
   // Wrong: Undefined + 1 = NaN
   counts[bird]++;
   
   // Right: Initialize first
   counts[bird] = (counts[bird] || 0) + 1;
   ```

## 🔄 Variations

```javascript
// Find k most common bird types
function topKBirds(birds, k) {
    const counts = {};
    for (const bird of birds) {
        counts[bird] = (counts[bird] || 0) + 1;
    }
    
    return Object.entries(counts)
        .sort((a, b) => b[1] - a[1] || a[0] - b[0])
        .slice(0, k)
        .map(([type]) => Number(type));
}

// Find least common bird type
function leastCommonBird(birds) {
    const counts = new Map();
    
    for (const bird of birds) {
        counts.set(bird, (counts.get(bird) || 0) + 1);
    }
    
    let minCount = Infinity;
    let leastCommon = 1;
    
    for (let type = 1; type <= 5; type++) {
        const count = counts.get(type) || 0;
        if (count > 0 && count < minCount) {
            minCount = count;
            leastCommon = type;
        }
    }
    
    return leastCommon;
}

// Return all types with maximum frequency
function allMostCommonBirds(birds) {
    const counts = [0, 0, 0, 0, 0];
    
    for (const bird of birds) {
        counts[bird - 1]++;
    }
    
    const maxCount = Math.max(...counts);
    const result = [];
    
    for (let i = 0; i < 5; i++) {
        if (counts[i] === maxCount) {
            result.push(i + 1);
        }
    }
    
    return result;
}
```

## 🔗 Related Problems

- **Most Common Element** - General frequency counting
- **Top K Frequent Elements** - Finding multiple frequent items
- **Majority Element** - Element appearing > n/2 times
- **Mode in BST** - Finding mode in tree structure

## 💡 Key Takeaways

1. **Frequency Counting**: Classic use case for hash maps
2. **Fixed Range Optimization**: Use array when types are limited
3. **Tie-Breaking Rules**: Process in specific order for deterministic results
4. **Space-Time Tradeoff**: O(1) space possible with fixed types

This problem teaches frequency analysis and the importance of carefully handling edge cases like ties!
