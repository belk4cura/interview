# Container With Most Water

## 🎯 What Are We Trying To Solve?

Find the maximum amount of water that can be contained between two vertical lines, where the water level is limited by the shorter line.

## 📝 The Problem

Given an array `height` where `height[i]` represents the height of a vertical line at position `i`, find two lines that together with the x-axis form a container that holds the most water.

```
Example 1:
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49
Explanation: Lines at index 1 (height=8) and index 8 (height=7) form the max area = 7 * 7 = 49

Example 2:
Input: height = [1,1]
Output: 1
```

## 💻 The Code

```javascript
/**
 * Container With Most Water - Two Pointer Solution
 * Time: O(n), Space: O(1)
 */
function maxArea(height) {
    let maxWater = 0;
    let left = 0;
    let right = height.length - 1;
    
    while (left < right) {
        // Calculate current water area
        const width = right - left;
        const minHeight = Math.min(height[left], height[right]);
        const currentWater = width * minHeight;
        
        // Update maximum water found
        maxWater = Math.max(maxWater, currentWater);
        
        // Move the pointer with smaller height
        // (moving the larger one can't increase area)
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    
    return maxWater;
}

/**
 * Optimized with early termination
 * Time: O(n), Space: O(1)
 */
function maxAreaOptimized(height) {
    let maxWater = 0;
    let left = 0;
    let right = height.length - 1;
    
    while (left < right) {
        // Calculate area with current boundaries
        const width = right - left;
        const leftHeight = height[left];
        const rightHeight = height[right];
        const minHeight = Math.min(leftHeight, rightHeight);
        const currentWater = width * minHeight;
        
        maxWater = Math.max(maxWater, currentWater);
        
        // Skip elements that can't possibly give better result
        if (leftHeight < rightHeight) {
            const prevHeight = leftHeight;
            while (left < right && height[left] <= prevHeight) {
                left++;
            }
        } else {
            const prevHeight = rightHeight;
            while (left < right && height[right] <= prevHeight) {
                right--;
            }
        }
    }
    
    return maxWater;
}

/**
 * Brute Force Solution (for comparison)
 * Time: O(n²), Space: O(1)
 */
function maxAreaBruteForce(height) {
    let maxWater = 0;
    
    for (let i = 0; i < height.length - 1; i++) {
        for (let j = i + 1; j < height.length; j++) {
            const width = j - i;
            const minHeight = Math.min(height[i], height[j]);
            const water = width * minHeight;
            maxWater = Math.max(maxWater, water);
        }
    }
    
    return maxWater;
}

// Test cases
console.log(maxArea([1,8,6,2,5,4,8,3,7])); // 49
console.log(maxArea([1,1])); // 1
console.log(maxArea([4,3,2,1,4])); // 16
console.log(maxArea([1,2,1])); // 2
console.log(maxArea([2,1,5,6,2,3])); // 10
```

## 🎨 The Approach

### Two-Pointer Strategy:
1. **Start at extremes** - Maximum width initially
2. **Calculate area** - width × min(left_height, right_height)
3. **Move intelligently** - Always move the pointer with smaller height
4. **Track maximum** - Keep best area found

### Visual Walkthrough:
```
height = [1,8,6,2,5,4,8,3,7]
         0 1 2 3 4 5 6 7 8

Initial state:
    8|   █
    7|   █           █
    6|   █ █         █
    5|   █ █   █     █
    4|   █ █   █ █   █
    3|   █ █   █ █ █ █
    2|   █ █ █ █ █ █ █
    1| █ █ █ █ █ █ █ █ █
      ─────────────────────
      L               R
      
Area = (8-0) × min(1,7) = 8 × 1 = 8

Move left pointer (smaller height):
      L → 
      Area = (8-1) × min(8,7) = 7 × 7 = 49 ✓
```

## 🤔 Why This Works

- **Greedy choice**: Moving the pointer with larger height can never increase area
- **Width decreases**: As pointers move inward, we need taller lines to compensate
- **Optimal substructure**: Best container must have at least one boundary at an optimal position

## ⚠️ Common Pitfalls

1. **Moving wrong pointer** - Must move the shorter line
2. **Integer overflow** - For large arrays with large heights
3. **Off-by-one errors** - Width calculation should be `right - left`
4. **Not considering width** - Area depends on both height AND width

## 🎯 Key Insights

- **Why move shorter line?**
  - Current area is limited by shorter line
  - Moving longer line guarantees smaller or equal area
  - Moving shorter line might find taller line

- **Proof of correctness**:
  - We never miss the optimal solution
  - If optimal is (i, j), we'll evaluate it before passing it

## 📊 Complexity Analysis

- **Time**: O(n) - Single pass with two pointers
- **Space**: O(1) - Only using constant extra variables
- **Optimal**: Can't do better than O(n) as we need to see all elements

## 🔄 Related Problems

- **Trapping Rain Water** - Similar but accumulates water
- **Largest Rectangle in Histogram** - Height constraints differ
- **Two Sum** - Basic two-pointer pattern
- **3Sum** - Extended two-pointer technique
