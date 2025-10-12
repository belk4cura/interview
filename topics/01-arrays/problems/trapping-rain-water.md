# Trapping Rain Water

## 🎯 What Are We Trying To Solve?

Calculating trapped rainwater is like measuring how much water would collect between buildings after rain - we need to find the "valleys" where water can accumulate.

## 📝 The Problem

Given an elevation map represented by an array, calculate how much rainwater can be trapped after raining.

```
Example:
height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

Visual:
     #
   # ### #
 # ### #####
[0,1,0,2,1,0,1,3,2,1,2,1]
   w   w w w     w w     = 6 units
```

## 💻 The Code

```javascript
/**
 * Two Pointers Solution
 * Time: O(n), Space: O(1)
 */
function trap(height) {
    let left = 0, right = height.length - 1;
    let leftMax = 0, rightMax = 0;
    let water = 0;
    
    while (left < right) {
        if (height[left] < height[right]) {
            if (height[left] >= leftMax) {
                leftMax = height[left];
            } else {
                water += leftMax - height[left];
            }
            left++;
        } else {
            if (height[right] >= rightMax) {
                rightMax = height[right];
            } else {
                water += rightMax - height[right];
            }
            right--;
        }
    }
    
    return water;
}

/**
 * Dynamic Programming Solution
 * Time: O(n), Space: O(n)
 */
function trapDP(height) {
    const n = height.length;
    if (n === 0) return 0;
    
    const leftMax = new Array(n);
    const rightMax = new Array(n);
    
    leftMax[0] = height[0];
    for (let i = 1; i < n; i++) {
        leftMax[i] = Math.max(leftMax[i - 1], height[i]);
    }
    
    rightMax[n - 1] = height[n - 1];
    for (let i = n - 2; i >= 0; i--) {
        rightMax[i] = Math.max(rightMax[i + 1], height[i]);
    }
    
    let water = 0;
    for (let i = 0; i < n; i++) {
        water += Math.min(leftMax[i], rightMax[i]) - height[i];
    }
    
    return water;
}
```

## 🎨 Key Insights

- **Water Level**: Min(leftMax, rightMax) - currentHeight
- **Two Pointers**: Move from lower height side
- **Precompute Max**: DP stores max heights for efficiency

## ⏱️ Complexity

- Two Pointers: O(n) time, O(1) space
- DP: O(n) time, O(n) space
