# Missing Numbers

## 🎯 What Are We Trying To Solve?

You have two arrays where the second array is a subset of the first (some numbers were removed). Find which numbers are missing from the second array. It's like comparing two attendance lists to see who's absent!

## 📝 The Problem

Given two arrays where the second array `arr` is created by removing some elements from the first array `brr`, find all the missing numbers. The difference between max and min values in the arrays is less than or equal to 100.

```
Example:
Original: [7, 2, 5, 3, 5, 3]
After removal: [7, 2, 5, 3]

Missing: [3, 5] (one 3 and one 5 were removed)
```

## 📊 Examples

**Example 1:**
```
Input: 
arr = [7, 2, 5, 3]
brr = [7, 2, 5, 3, 5, 3]

Output: [3, 5]

Explanation:
brr has: 7(×1), 2(×1), 5(×2), 3(×2)
arr has: 7(×1), 2(×1), 5(×1), 3(×1)
Missing: 5(×1), 3(×1)
```

**Example 2:**
```
Input:
arr = [1, 2, 3, 4, 5]
brr = [1, 2, 3, 4, 5, 6, 7]

Output: [6, 7]
```

## 💻 The Code

```javascript
/**
 * Find missing numbers from arr that exist in brr
 * @param {number[]} arr - Array with missing numbers
 * @param {number[]} brr - Original complete array
 * @returns {number[]} - Missing numbers in ascending order
 */
function missingNumbers(arr, brr) {
    // Count frequencies in both arrays
    const freqArr = new Map();
    const freqBrr = new Map();
    
    // Count frequencies in brr (original)
    for (const num of brr) {
        freqBrr.set(num, (freqBrr.get(num) || 0) + 1);
    }
    
    // Count frequencies in arr (with missing)
    for (const num of arr) {
        freqArr.set(num, (freqArr.get(num) || 0) + 1);
    }
    
    // Find missing numbers
    const missing = [];
    for (const [num, count] of freqBrr) {
        const arrCount = freqArr.get(num) || 0;
        if (arrCount < count) {
            missing.push(num);
        }
    }
    
    // Return sorted array
    return missing.sort((a, b) => a - b);
}

// Alternative: Optimized for constraint (range ≤ 100)
function missingNumbersOptimized(arr, brr) {
    // Find min value to use as offset
    const min = Math.min(...brr);
    
    // Create frequency array (max range is 101)
    const frequency = new Array(101).fill(0);
    
    // Increment for each number in brr
    for (const num of brr) {
        frequency[num - min]++;
    }
    
    // Decrement for each number in arr
    for (const num of arr) {
        frequency[num - min]--;
    }
    
    // Collect missing numbers
    const missing = [];
    for (let i = 0; i < frequency.length; i++) {
        if (frequency[i] > 0) {
            missing.push(i + min);
        }
    }
    
    return missing;
}

// Alternative: Single pass with difference tracking
function missingNumbersSinglePass(arr, brr) {
    const diff = new Map();
    
    // Add all from brr
    for (const num of brr) {
        diff.set(num, (diff.get(num) || 0) + 1);
    }
    
    // Remove all from arr
    for (const num of arr) {
        const count = diff.get(num);
        if (count === 1) {
            diff.delete(num);
        } else {
            diff.set(num, count - 1);
        }
    }
    
    // Return remaining numbers sorted
    return Array.from(diff.keys()).sort((a, b) => a - b);
}

// Alternative: Using object for counting
function missingNumbersObject(arr, brr) {
    const counts = {};
    
    // Count occurrences in brr
    brr.forEach(num => {
        counts[num] = (counts[num] || 0) + 1;
    });
    
    // Subtract occurrences in arr
    arr.forEach(num => {
        counts[num]--;
    });
    
    // Find numbers with positive count
    return Object.keys(counts)
        .filter(num => counts[num] > 0)
        .map(Number)
        .sort((a, b) => a - b);
}

// Test cases
console.log(missingNumbers([7, 2, 5, 3], [7, 2, 5, 3, 5, 3])); // [3, 5]
console.log(missingNumbers([1, 2, 3, 4, 5], [1, 2, 3, 4, 5, 6, 7])); // [6, 7]
console.log(missingNumbers([203, 204, 205], [203, 204, 204, 205, 206, 207])); // [204, 206, 207]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `missingNumbers([7, 2, 5, 3], [7, 2, 5, 3, 5, 3])`:

```
Step 1: Count frequencies in brr (original)
brr = [7, 2, 5, 3, 5, 3]
freqBrr = {
    7: 1,
    2: 1,
    5: 2,  ← appears twice
    3: 2   ← appears twice
}

Step 2: Count frequencies in arr (with missing)
arr = [7, 2, 5, 3]
freqArr = {
    7: 1,
    2: 1,
    5: 1,  ← only once (one is missing!)
    3: 1   ← only once (one is missing!)
}

Step 3: Find differences
For each number in brr:
- 7: brr(1) - arr(1) = 0 (not missing)
- 2: brr(1) - arr(1) = 0 (not missing)
- 5: brr(2) - arr(1) = 1 (missing!)
- 3: brr(2) - arr(1) = 1 (missing!)

Result: [3, 5] (sorted)
```

## 📊 Visual Understanding

```
Optimized Array Approach (range ≤ 100):

Numbers: 203, 204, 204, 205, 206, 207
Min: 203

Index mapping:
203 → index 0
204 → index 1
205 → index 2
206 → index 3
207 → index 4

Frequency array after processing brr:
[1, 2, 1, 1, 1]
 ↑  ↑  ↑  ↑  ↑
203 204 205 206 207

After removing arr [203, 204, 205]:
[0, 1, 0, 1, 1]
    ↑     ↑  ↑
   204   206 207 (missing)
```

## ⏱️ Complexity Analysis

**Map/Object Solution:**
- Time: O(n + m) where n and m are array lengths
- Space: O(k) where k is unique numbers

**Array Solution (with constraint):**
- Time: O(n + m)
- Space: O(100) = O(1) constant due to constraint

## 🚫 Common Mistakes

1. **Not Handling Duplicates**
   ```javascript
   // Wrong: Only checking existence
   const set1 = new Set(arr);
   const set2 = new Set(brr);
   // This misses duplicate removals!
   
   // Right: Count frequencies
   const freq = new Map();
   ```

2. **Forgetting to Sort Output**
   ```javascript
   // Wrong: Returning unsorted
   return Array.from(missing);
   
   // Right: Sort before returning
   return Array.from(missing).sort((a, b) => a - b);
   ```

3. **Off-by-One in Array Method**
   ```javascript
   // Wrong: Array size too small
   const frequency = new Array(100);  // Should be 101
   
   // Right: Account for inclusive range
   const frequency = new Array(101);
   ```

## 🔄 Variations

```javascript
// Find all differences (both added and removed)
function findDifferences(arr1, arr2) {
    const freq1 = new Map();
    const freq2 = new Map();
    
    arr1.forEach(n => freq1.set(n, (freq1.get(n) || 0) + 1));
    arr2.forEach(n => freq2.set(n, (freq2.get(n) || 0) + 1));
    
    const result = {
        missing: [],  // in arr1 but not arr2
        added: []     // in arr2 but not arr1
    };
    
    // Check missing from arr1
    freq1.forEach((count, num) => {
        const count2 = freq2.get(num) || 0;
        if (count > count2) {
            for (let i = 0; i < count - count2; i++) {
                result.missing.push(num);
            }
        }
    });
    
    // Check added to arr2
    freq2.forEach((count, num) => {
        const count1 = freq1.get(num) || 0;
        if (count > count1) {
            for (let i = 0; i < count - count1; i++) {
                result.added.push(num);
            }
        }
    });
    
    return result;
}

// Find first k missing numbers
function firstKMissing(arr, brr, k) {
    const missing = missingNumbers(arr, brr);
    return missing.slice(0, k);
}
```

## 🔗 Related Problems

- **Find All Duplicates** - Finding repeated elements
- **Find the Difference** - Single character difference
- **Set Mismatch** - Find duplicate and missing
- **First Missing Positive** - Find smallest missing positive

## 💡 Key Takeaways

1. **Frequency Counting**: Essential for handling duplicates
2. **Space Optimization**: Use arrays when range is limited
3. **Hash Maps vs Arrays**: Trade-off between flexibility and efficiency
4. **Sort Requirements**: Many problems expect sorted output

This problem demonstrates the importance of choosing the right data structure based on constraints!
