# Colorful Number

## 🎯 What Are We Trying To Solve?

A number is called "colorful" if all products of consecutive subsequences of its digits are different. Imagine each product as a different color - a colorful number has no repeated colors! It's like ensuring every substring of digits creates a unique multiplication fingerprint.

## 📝 The Problem

Given a number, determine if it's colorful. For a colorful number, products of every consecutive subsequence of digits are different.

```
Example:
Number: 236

Subsequences and their products:
- Single digits: 2 (2), 3 (3), 6 (6)
- Pairs: 23 (2×3=6), 36 (3×6=18) 
- All: 236 (2×3×6=36)

Products: [2, 3, 6, 6, 18, 36]
          ↑     ↑ Duplicate!
Not colorful because 6 appears twice
```

## 📊 Examples

**Example 1:**
```
Input: 3245
Output: true

Products:
- Singles: 3, 2, 4, 5
- Pairs: 3×2=6, 2×4=8, 4×5=20
- Triples: 3×2×4=24, 2×4×5=40
- All: 3×2×4×5=120

All unique: [3,2,4,5,6,8,20,24,40,120] ✓
```

**Example 2:**
```
Input: 326
Output: false

Products:
- Singles: 3, 2, 6
- Pairs: 3×2=6, 2×6=12
- All: 3×2×6=36

Duplicate found: 6 appears as single digit and as product of 3×2
```

## 💻 The Code

```javascript
/**
 * Check if a number is colorful
 * @param {number} num - The number to check
 * @returns {boolean} - True if colorful, false otherwise
 */
function isColorful(num) {
    // Convert number to array of digits
    const digits = [];
    let temp = num;
    
    while (temp > 0) {
        digits.unshift(temp % 10);  // Add to front to maintain order
        temp = Math.floor(temp / 10);
    }
    
    // Set to store all products
    const products = new Set();
    
    // Check all consecutive subsequences
    for (let length = 1; length <= digits.length; length++) {
        for (let start = 0; start <= digits.length - length; start++) {
            // Calculate product of subsequence
            let product = 1;
            for (let i = start; i < start + length; i++) {
                product *= digits[i];
            }
            
            // Check if product already exists
            if (products.has(product)) {
                return false;  // Duplicate found - not colorful
            }
            
            products.add(product);
        }
    }
    
    return true;  // All products unique - colorful!
}

// Alternative: Cleaner implementation
function isColorfulClean(num) {
    // Convert to string for easier digit access
    const digitStr = num.toString();
    const products = new Set();
    
    // Check all substrings
    for (let i = 0; i < digitStr.length; i++) {
        let product = 1;
        
        for (let j = i; j < digitStr.length; j++) {
            product *= parseInt(digitStr[j]);
            
            if (products.has(product)) {
                return false;
            }
            products.add(product);
        }
    }
    
    return true;
}

// Alternative: Using sliding window
function isColorfulSlidingWindow(num) {
    const digits = num.toString().split('').map(Number);
    const products = new Set();
    
    // For each window size
    for (let windowSize = 1; windowSize <= digits.length; windowSize++) {
        // Slide the window
        for (let start = 0; start <= digits.length - windowSize; start++) {
            const end = start + windowSize;
            const subsequence = digits.slice(start, end);
            const product = subsequence.reduce((a, b) => a * b, 1);
            
            if (products.has(product)) {
                return false;
            }
            products.add(product);
        }
    }
    
    return true;
}

// Test cases
console.log(isColorful(3245));  // true
console.log(isColorful(326));   // false
console.log(isColorful(236));   // false (2×3 = 6, and 6 is a digit)
console.log(isColorful(23));    // true
console.log(isColorful(1));     // true
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `isColorful(236)`:

```
Number: 236
Digits: [2, 3, 6]

Step 1: Check subsequences of length 1
- Start 0: digits[0] = 2, product = 2
  products = {2} ✓
- Start 1: digits[1] = 3, product = 3
  products = {2, 3} ✓
- Start 2: digits[2] = 6, product = 6
  products = {2, 3, 6} ✓

Step 2: Check subsequences of length 2
- Start 0: digits[0,1] = [2,3], product = 2×3 = 6
  6 already in products! Return false ✗

The number is NOT colorful because 6 appears both as:
- A single digit
- The product of 2×3
```

## 📊 Visual Understanding

```
Number: 3245

All Subsequences:
Length 1: [3] [2] [4] [5]
          ↓   ↓   ↓   ↓
          3   2   4   5

Length 2: [3,2] [2,4] [4,5]
          ↓     ↓     ↓
          6     8     20

Length 3: [3,2,4] [2,4,5]
          ↓       ↓
          24      40

Length 4: [3,2,4,5]
          ↓
          120

Products Set: {3,2,4,5,6,8,20,24,40,120}
All unique → Colorful! ✓
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(n²) where n is the number of digits
  - Two nested loops for all subsequences
  - Product calculation is O(n) worst case
  - Total: O(n³) but typically O(n²) for small digit counts
  
- **Space Complexity:** O(n²)
  - Maximum n(n+1)/2 different products to store

## 🚫 Common Mistakes

1. **Not Checking All Subsequences**
   ```javascript
   // Wrong: Only checking adjacent pairs
   for (let i = 0; i < digits.length - 1; i++) {
       const product = digits[i] * digits[i + 1];
   }
   
   // Right: Check all consecutive subsequences
   for (let length = 1; length <= digits.length; length++) {
       for (let start = 0; start <= digits.length - length; start++) {
   ```

2. **Integer Overflow for Large Numbers**
   ```javascript
   // Wrong: Processing the entire number
   while (num > 0) {
       // May overflow for very large numbers
   }
   
   // Right: Convert to string first if needed
   const digitStr = num.toString();
   ```

3. **Not Handling Edge Cases**
   ```javascript
   // Add edge case handling
   if (num < 10) return true;  // Single digit is always colorful
   if (num.toString().includes('0') || num.toString().includes('1')) {
       // Numbers with 0 or 1 need special handling
       // 0 × anything = 0 (duplicates)
       // 1 × n = n (might create duplicates)
   }
   ```

## 🔄 Variations

```javascript
// Check if colorful ignoring 0 and 1
function isColorfulIgnoreSpecial(num) {
    const digitStr = num.toString().replace(/[01]/g, '');
    if (digitStr.length === 0) return true;
    
    return isColorfulClean(parseInt(digitStr));
}

// Find all colorful numbers in range
function findColorfulNumbers(start, end) {
    const colorful = [];
    
    for (let num = start; num <= end; num++) {
        if (isColorful(num)) {
            colorful.push(num);
        }
    }
    
    return colorful;
}

// Get colorful score (number of unique products)
function colorfulScore(num) {
    const digits = num.toString().split('').map(Number);
    const products = new Set();
    
    for (let i = 0; i < digits.length; i++) {
        let product = 1;
        for (let j = i; j < digits.length; j++) {
            product *= digits[j];
            products.add(product);
        }
    }
    
    return products.size;
}
```

## 🔗 Related Problems

- **Product of Array Except Self** - Array products
- **Subarray Product Less Than K** - Product constraints
- **Maximum Product Subarray** - Finding optimal products
- **Happy Number** - Number property checking

## 💡 Key Takeaways

1. **Set for Uniqueness**: Perfect data structure for checking duplicates
2. **Nested Loops for Subsequences**: Standard pattern for all substrings/subarrays
3. **Early Exit**: Return false as soon as duplicate is found
4. **String vs Number**: Sometimes easier to work with string representation

This problem combines number manipulation, subsequence generation, and hash map usage - a great exercise in problem decomposition!
