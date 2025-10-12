# Search in 2D Matrix

## 🎯 What Are We Trying To Solve?

Imagine you have a spreadsheet where numbers increase as you go right across each row AND increase as you go down each column. You need to find if a specific number exists in this spreadsheet. It's like looking for a book in a library where books are sorted by title horizontally and by author vertically!

## 📝 The Problem

Write an efficient algorithm to search for a value in an m × n matrix. This matrix has the following properties:
- Integers in each row are sorted from left to right
- Integers in each column are sorted from top to bottom

```
Example Matrix:
[
  [1,  4,  7,  11, 15],
  [2,  5,  8,  12, 19],
  [3,  6,  9,  16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]

Target = 5 → true
Target = 20 → false
```

## 🧠 How to Think About This

Start from the top-right corner (or bottom-left). From there, you can eliminate either an entire row or column with each comparison!

### Visual Strategy
```
Start at top-right (15):
- If target < 15: Move left (smaller values)
- If target > 15: Move down (larger values)
- If target = 15: Found!

This creates a "staircase" search pattern!
```

## 💻 The Code

```javascript
/**
 * Search in row-wise and column-wise sorted matrix
 * Time: O(m + n) where m = rows, n = columns
 * Space: O(1) - only using two pointers
 */
function searchMatrix(matrix, target) {
    // Handle empty matrix
    if (!matrix.length || !matrix[0].length) {
        return false;
    }
    
    // Start from top-right corner
    // Why? It's the largest in its row and smallest in its column!
    let row = 0;
    let col = matrix[0].length - 1;
    
    // Continue while within bounds
    while (row < matrix.length && col >= 0) {
        const current = matrix[row][col];
        
        if (current === target) {
            return true;  // Found it!
        }
        
        if (current > target) {
            // Current is too large
            // All values below are even larger (column sorted)
            // So move left to smaller values
            col--;
        } else {
            // Current is too small
            // All values to the left are even smaller (row sorted)
            // So move down to larger values
            row++;
        }
    }
    
    return false;  // Exhausted search space
}

// Test cases
const matrix = [
    [1,  4,  7,  11, 15],
    [2,  5,  8,  12, 19],
    [3,  6,  9,  16, 22],
    [10, 13, 14, 17, 24],
    [18, 21, 23, 26, 30]
];

console.log(searchMatrix(matrix, 5));   // true
console.log(searchMatrix(matrix, 20));  // false
console.log(searchMatrix(matrix, 11));  // true
```

## 🎨 Step-by-Step Walkthrough

Let's search for target = 5:

```
Matrix:                    Start Position (row=0, col=4):
[1,  4,  7,  11, 15] ←    Value = 15
[2,  5,  8,  12, 19]
[3,  6,  9,  16, 22]
[10, 13, 14, 17, 24]
[18, 21, 23, 26, 30]

Step 1: Compare 15 with 5
- 15 > 5, move left
- row=0, col=3, value=11

Step 2: Compare 11 with 5
- 11 > 5, move left
- row=0, col=2, value=7

Step 3: Compare 7 with 5
- 7 > 5, move left
- row=0, col=1, value=4

Step 4: Compare 4 with 5
- 4 < 5, move down
- row=1, col=1, value=5

Step 5: Compare 5 with 5
- 5 = 5, FOUND! Return true
```

## 📊 Why O(m + n)?

### The Staircase Pattern

```
Worst case path (searching for non-existent element):
[S, ←, ←, ←, ←]    S = Start
[↓,  ,  ,  ,  ]    ← = Move left
[↓,  ,  ,  ,  ]    ↓ = Move down
[↓,  ,  ,  ,  ]
[E,  ,  ,  ,  ]    E = End

Maximum steps = (n-1) left moves + (m-1) down moves = m + n - 2 = O(m + n)
```

### Why Not Binary Search?

```javascript
// You might think: binary search each row = O(m log n)
for (let row = 0; row < matrix.length; row++) {
    if (binarySearch(matrix[row], target)) return true;
}
// This works but is slower! O(m log n) vs O(m + n)
```

## 🚫 Common Mistakes

1. **Starting from Wrong Corner**
   ```javascript
   // Wrong: Top-left doesn't give clear direction
   let row = 0, col = 0;
   // If current < target, should we go right or down? Ambiguous!
   
   // Right: Top-right or bottom-left
   let row = 0, col = matrix[0].length - 1;  // Top-right
   // OR
   let row = matrix.length - 1, col = 0;     // Bottom-left
   ```

2. **Not Checking Bounds**
   ```javascript
   // Wrong: Can go out of bounds
   while (true) {
       if (matrix[row][col] === target) return true;
       // Missing bounds check!
   }
   
   // Right: Check bounds in loop condition
   while (row < matrix.length && col >= 0)
   ```

3. **Wrong Movement Direction**
   ```javascript
   // Wrong: Moving in wrong direction
   if (current > target) {
       row++;  // Wrong! Should move to smaller values
   }
   ```

## 🔄 Variations

### Variation 1: Count Occurrences
```javascript
function countTarget(matrix, target) {
    let count = 0;
    let row = 0;
    let col = matrix[0].length - 1;
    
    while (row < matrix.length && col >= 0) {
        if (matrix[row][col] === target) {
            count++;
            // Continue searching! 
            // Move both down and left to check for more
            let tempCol = col - 1;
            while (tempCol >= 0 && matrix[row][tempCol] === target) {
                count++;
                tempCol--;
            }
            row++;
        } else if (matrix[row][col] > target) {
            col--;
        } else {
            row++;
        }
    }
    
    return count;
}
```

### Variation 2: Find K-th Smallest
```javascript
function kthSmallest(matrix, k) {
    const n = matrix.length;
    let low = matrix[0][0];
    let high = matrix[n-1][n-1];
    
    while (low < high) {
        const mid = Math.floor((low + high) / 2);
        const count = countLessOrEqual(matrix, mid);
        
        if (count < k) {
            low = mid + 1;
        } else {
            high = mid;
        }
    }
    
    return low;
}

function countLessOrEqual(matrix, target) {
    let count = 0;
    let row = matrix.length - 1;
    let col = 0;
    
    while (row >= 0 && col < matrix[0].length) {
        if (matrix[row][col] <= target) {
            count += row + 1;  // All elements above in this column
            col++;
        } else {
            row--;
        }
    }
    
    return count;
}
```

## 🔗 Related Problems

- **Search a 2D Matrix** - Fully sorted matrix (treat as 1D array)
- **Kth Smallest Element in Sorted Matrix** - Uses similar traversal
- **Leftmost Column with at Least a One** - Binary matrix search
- **Find Negative Numbers in Sorted Matrix** - Count negatives

## 💡 Key Takeaways

1. **Choose Strategic Starting Point**: Top-right or bottom-left corners
2. **Eliminate Row/Column**: Each comparison eliminates entire row or column
3. **Linear Time**: O(m + n) beats binary search on each row
4. **Staircase Pattern**: Visualize the search path as stairs

This technique is powerful for any matrix with sorted rows and columns!
