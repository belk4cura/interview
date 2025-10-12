# Move Zeroes

## 🎯 What Are We Trying To Solve?

Move all zeros in an array to the end while maintaining the relative order of non-zero elements, doing this in-place.

## 📝 The Problem

Given an integer array `nums`, move all 0's to the end of it while maintaining the relative order of the non-zero elements. Must be done in-place without making a copy of the array.

```
Example 1:
Input: nums = [0,1,0,3,12]
Output: [1,3,12,0,0]

Example 2:
Input: nums = [0]
Output: [0]

Example 3:
Input: nums = [1,2,3]
Output: [1,2,3]
```

## 💻 The Code

```javascript
/**
 * Move Zeroes - Two Pointer Solution
 * Time: O(n), Space: O(1)
 */
function moveZeroes(nums) {
    let nonZeroPos = 0; // Position for next non-zero element
    
    // Move all non-zero elements to the front
    for (let i = 0; i < nums.length; i++) {
        if (nums[i] !== 0) {
            nums[nonZeroPos] = nums[i];
            nonZeroPos++;
        }
    }
    
    // Fill remaining positions with zeros
    for (let i = nonZeroPos; i < nums.length; i++) {
        nums[i] = 0;
    }
    
    return nums;
}

/**
 * Optimized with Swapping - Fewer writes
 * Time: O(n), Space: O(1)
 */
function moveZeroesSwap(nums) {
    let nonZeroPos = 0;
    
    for (let i = 0; i < nums.length; i++) {
        if (nums[i] !== 0) {
            // Swap only if positions are different
            if (i !== nonZeroPos) {
                [nums[nonZeroPos], nums[i]] = [nums[i], nums[nonZeroPos]];
            }
            nonZeroPos++;
        }
    }
    
    return nums;
}

/**
 * Snowball approach - Accumulate zeros
 * Time: O(n), Space: O(1)
 */
function moveZeroesSnowball(nums) {
    let snowballSize = 0;
    
    for (let i = 0; i < nums.length; i++) {
        if (nums[i] === 0) {
            snowballSize++;
        } else if (snowballSize > 0) {
            // Move current element back by snowball size
            nums[i - snowballSize] = nums[i];
            nums[i] = 0;
        }
    }
    
    return nums;
}

/**
 * Generic: Move all X to end
 * Time: O(n), Space: O(1)
 */
function moveToEnd(nums, target) {
    let writePos = 0;
    
    // Write all non-target elements
    for (let i = 0; i < nums.length; i++) {
        if (nums[i] !== target) {
            nums[writePos] = nums[i];
            writePos++;
        }
    }
    
    // Fill rest with target
    for (let i = writePos; i < nums.length; i++) {
        nums[i] = target;
    }
    
    return nums;
}

/**
 * Move zeros to front instead
 * Time: O(n), Space: O(1)
 */
function moveZeroesToFront(nums) {
    let writePos = nums.length - 1;
    
    // Traverse from end, place non-zeros from back
    for (let i = nums.length - 1; i >= 0; i--) {
        if (nums[i] !== 0) {
            nums[writePos] = nums[i];
            writePos--;
        }
    }
    
    // Fill front with zeros
    for (let i = 0; i <= writePos; i++) {
        nums[i] = 0;
    }
    
    return nums;
}

// Test cases
console.log(moveZeroes([0,1,0,3,12])); // [1,3,12,0,0]
console.log(moveZeroesSwap([0,0,1])); // [1,0,0]
console.log(moveZeroesSnowball([1,0,2,0,3])); // [1,2,3,0,0]
console.log(moveToEnd([1,2,1,3,1], 1)); // [2,3,1,1,1]
console.log(moveZeroesToFront([0,1,0,3,12])); // [0,0,1,3,12]
```

## 🎨 The Approach

### Two-Pointer Strategy:
1. **Write pointer** - Tracks position for next non-zero
2. **Read pointer** - Scans through array
3. **Copy non-zeros** - Place at write position
4. **Fill zeros** - Fill remaining with zeros

### Visual Walkthrough:
```
Initial: [0,1,0,3,12]
         w,r

Step 1: nums[0]=0, skip
        [0,1,0,3,12]
         w r

Step 2: nums[1]=1≠0, write at w=0
        [1,1,0,3,12]
           w r

Step 3: nums[2]=0, skip
        [1,1,0,3,12]
           w   r

Step 4: nums[3]=3≠0, write at w=1
        [1,3,0,3,12]
             w   r

Step 5: nums[4]=12≠0, write at w=2
        [1,3,12,3,12]
                w    r

Fill zeros from w to end:
        [1,3,12,0,0]
```

### Swap Approach Walkthrough:
```
[0,1,0,3,12]
 w r          nums[0]=0, w stays

[0,1,0,3,12]
 w   r        nums[1]=1≠0, swap with w
[1,0,0,3,12]
   w   r      

[1,0,0,3,12]
   w     r    nums[3]=3≠0, swap with w
[1,3,0,0,12]
     w     r  

[1,3,0,0,12]
     w       r  nums[4]=12≠0, swap with w
[1,3,12,0,0]
```

## 🤔 Why This Works

- **Stable partitioning**: Maintains relative order of non-zeros
- **Two-pass simplicity**: First pass moves non-zeros, second fills zeros
- **In-place efficiency**: No extra array needed
- **Single scan possible**: Swap approach does it in one pass

## ⚠️ Common Pitfalls

1. **Changing relative order** - Non-zeros must stay in original order
2. **Not in-place** - Creating new array violates constraint
3. **Unnecessary swaps** - Swapping when indices are same
4. **Index bounds** - Careful with array boundaries

## 🎯 Key Insights

### Two approaches comparison:
1. **Overwrite + Fill**: Simple, two passes, more writes
2. **Swap**: One pass, fewer writes, slightly complex

### When to use each:
- **Overwrite**: When simplicity matters
- **Swap**: When minimizing writes is important

## 📊 Complexity Analysis

- **Time**: O(n) - Single or double pass through array
- **Space**: O(1) - In-place modification
- **Operations**: 
  - Overwrite: Up to 2n writes
  - Swap: Up to n swaps (2n reads/writes)

## 🔄 Related Problems

- **Remove Element** - Remove specific value
- **Remove Duplicates** - Keep unique elements
- **Sort Colors** - Dutch flag partitioning
- **Partition Array** - Split by condition
- **Segregate Even and Odd** - Similar partitioning
