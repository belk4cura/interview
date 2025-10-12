# Left Rotation

## 🎯 What Are We Trying To Solve?

Imagine you're standing in a line with your friends, and everyone needs to move left by a certain number of positions. The people at the front wrap around to the back. This is a left rotation! It's like a circular conveyor belt moving items to the left.

## 📝 The Problem

Given an array and a number `d`, perform `d` left rotations on the array. A left rotation moves the first element to the end of the array.

```
Example:
Array: [1, 2, 3, 4, 5]
Rotations: 2

After 1 rotation: [2, 3, 4, 5, 1]
After 2 rotations: [3, 4, 5, 1, 2]
```

## 📊 Examples

**Example 1:**
```
Input: arr = [1, 2, 3, 4, 5], d = 2
Output: [3, 4, 5, 1, 2]

Step by step:
Original: [1, 2, 3, 4, 5]
Rotate 1: [2, 3, 4, 5, 1]
Rotate 2: [3, 4, 5, 1, 2]
```

**Example 2:**
```
Input: arr = [1, 2, 3], d = 4
Output: [2, 3, 1]

Why? 4 rotations = 1 rotation (since 4 % 3 = 1)
After 3 rotations we're back to original
The 4th rotation is like doing 1 rotation
```

## 💻 The Code

```javascript
/**
 * Perform left rotation on an array
 * @param {number[]} arr - Array to rotate
 * @param {number} d - Number of rotations
 * @returns {number[]} - Rotated array
 */
function leftRotation(arr, d) {
    // Handle edge cases
    if (!arr || arr.length === 0 || d === 0) {
        return arr;
    }
    
    // Optimize: d rotations where d > length cycles back
    // Example: rotating array of 5 by 7 = rotating by 2
    const rotation = d % arr.length;
    
    // No rotation needed if it cycles back to original
    if (rotation === 0) return arr;
    
    // Method 1: Create new array with rotated elements
    const result = new Array(arr.length);
    
    for (let i = 0; i < arr.length; i++) {
        // Calculate new position after rotation
        // Element at index i moves to index (i - rotation)
        // Use modulo to handle wrap-around
        const newIndex = (i - rotation + arr.length) % arr.length;
        result[newIndex] = arr[i];
    }
    
    return result;
}

// Alternative: Using array slicing (more JavaScript-like)
function leftRotationSlice(arr, d) {
    if (!arr || arr.length === 0) return arr;
    
    const rotation = d % arr.length;
    if (rotation === 0) return arr;
    
    // Take elements from rotation to end, then from start to rotation
    return [...arr.slice(rotation), ...arr.slice(0, rotation)];
}

// Alternative: In-place rotation using reversal
function leftRotationInPlace(arr, d) {
    if (!arr || arr.length === 0) return arr;
    
    const n = arr.length;
    const rotation = d % n;
    if (rotation === 0) return arr;
    
    // Helper to reverse portion of array
    const reverse = (start, end) => {
        while (start < end) {
            [arr[start], arr[end]] = [arr[end], arr[start]];
            start++;
            end--;
        }
    };
    
    // Reverse first d elements
    reverse(0, rotation - 1);
    // Reverse remaining elements
    reverse(rotation, n - 1);
    // Reverse entire array
    reverse(0, n - 1);
    
    return arr;
}

// Test cases
console.log(leftRotation([1, 2, 3, 4, 5], 2));  // [3, 4, 5, 1, 2]
console.log(leftRotation([1, 2, 3], 4));        // [2, 3, 1]
console.log(leftRotation([1, 2, 3, 4, 5], 0));  // [1, 2, 3, 4, 5]
console.log(leftRotation([], 3));               // []
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `leftRotation([1, 2, 3, 4, 5], 2)`:

```
Initial: arr = [1, 2, 3, 4, 5], d = 2
rotation = 2 % 5 = 2

Creating new array with rotated positions:

i = 0: arr[0] = 1
  newIndex = (0 - 2 + 5) % 5 = 3
  result[3] = 1

i = 1: arr[1] = 2
  newIndex = (1 - 2 + 5) % 5 = 4
  result[4] = 2

i = 2: arr[2] = 3
  newIndex = (2 - 2 + 5) % 5 = 0
  result[0] = 3

i = 3: arr[3] = 4
  newIndex = (3 - 2 + 5) % 5 = 1
  result[1] = 4

i = 4: arr[4] = 5
  newIndex = (4 - 2 + 5) % 5 = 2
  result[2] = 5

Result: [3, 4, 5, 1, 2]
```

## 🔄 Visual Understanding

```
Original:     [1, 2, 3, 4, 5]
               ↑
               These 2 elements move to the end

After slicing:
First part:  arr.slice(2) = [3, 4, 5]
Second part: arr.slice(0, 2) = [1, 2]
Combined:    [3, 4, 5, 1, 2]

Or think of it as a circular array:
    1 → 2 → 3 → 4 → 5
    ↑               ↓
    ←←←←←←←←←←←←←←←←
    
Rotate left by 2:
    3 → 4 → 5 → 1 → 2
```

## ⏱️ Complexity Analysis

**Method 1 (New Array):**
- Time: O(n) - iterate once through array
- Space: O(n) - create new array

**Method 2 (Slice):**
- Time: O(n) - slicing creates new arrays
- Space: O(n) - creates intermediate arrays

**Method 3 (In-place Reversal):**
- Time: O(n) - three reversal passes
- Space: O(1) - modifies in place

## 🚫 Common Mistakes

1. **Not handling rotation > array length**
   ```javascript
   // Wrong: Rotating 5-element array 7 times
   for (let i = 0; i < 7; i++) {
       // Unnecessary iterations!
   }
   
   // Right: Use modulo
   const rotation = d % arr.length;  // 7 % 5 = 2
   ```

2. **Off-by-one in index calculation**
   ```javascript
   // Wrong: Forgetting to add array length
   const newIndex = (i - rotation) % arr.length;  // Can be negative!
   
   // Right: Add length to handle negative values
   const newIndex = (i - rotation + arr.length) % arr.length;
   ```

3. **Mutating original array unexpectedly**
   ```javascript
   // Wrong: Modifies input
   function rotate(arr, d) {
       arr.push(arr.shift());  // Mutates arr!
   }
   
   // Right: Return new array or clearly document mutation
   function rotate(arr, d) {
       const result = [...arr];  // Copy first
       // ... modify result
   }
   ```

## 🔄 Variations

```javascript
// Right rotation
function rightRotation(arr, d) {
    if (!arr || arr.length === 0) return arr;
    const rotation = d % arr.length;
    return [...arr.slice(-rotation), ...arr.slice(0, -rotation)];
}

// Rotate string
function rotateString(str, d) {
    const rotation = d % str.length;
    return str.slice(rotation) + str.slice(0, rotation);
}

// Rotate 2D matrix 90 degrees
function rotateMatrix(matrix) {
    const n = matrix.length;
    // Transpose
    for (let i = 0; i < n; i++) {
        for (let j = i; j < n; j++) {
            [matrix[i][j], matrix[j][i]] = [matrix[j][i], matrix[i][j]];
        }
    }
    // Reverse each row
    for (let i = 0; i < n; i++) {
        matrix[i].reverse();
    }
    return matrix;
}
```

## 🔗 Related Problems

- **Rotate Array** (LeetCode 189) - Same concept
- **Rotate String** - Check if one string is rotation of another
- **Circular Array Loop** - Detect cycles in circular array
- **Rotate Image** - 2D matrix rotation

## 💡 Key Takeaways

1. **Modulo Operation**: Essential for handling rotations > array length
2. **Multiple Approaches**: Trade-offs between space and simplicity
3. **Index Mapping**: Understanding how indices change after rotation
4. **Circular Nature**: Arrays can be thought of as circular for rotation

This is a fundamental array manipulation problem that appears in many forms!
