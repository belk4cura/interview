# Remove Duplicates from Sorted Array

## 🎯 What Are We Trying To Solve?

Remove duplicates from a sorted array in-place, returning the length of the modified array with unique elements.

## 📝 The Problem

Given a sorted array, remove duplicates in-place such that each element appears only once. Return the new length. The order must be preserved, and you must modify the array in-place with O(1) extra memory.

```
Example 1:
Input: nums = [1,1,2]
Output: 2, nums = [1,2,_]
Explanation: First 2 elements contain unique values

Example 2:
Input: nums = [0,0,1,1,1,2,2,3,3,4]
Output: 5, nums = [0,1,2,3,4,_,_,_,_,_]
```

## 💻 The Code

```javascript
/**
 * Remove Duplicates - Two Pointer Solution
 * Time: O(n), Space: O(1)
 */
function removeDuplicates(nums) {
    if (nums.length === 0) return 0;
    
    // Slow pointer tracks position for next unique element
    let slow = 0;
    
    // Fast pointer explores the array
    for (let fast = 1; fast < nums.length; fast++) {
        // Found a new unique element
        if (nums[fast] !== nums[slow]) {
            slow++;
            nums[slow] = nums[fast];
        }
        // If duplicate, just move fast pointer
    }
    
    // Length is slow index + 1
    return slow + 1;
}

/**
 * Remove Duplicates II - Allow at most 2 duplicates
 * Time: O(n), Space: O(1)
 */
function removeDuplicatesII(nums) {
    if (nums.length <= 2) return nums.length;
    
    // Position for next valid element
    let slow = 2;
    
    // Check from third element onwards
    for (let fast = 2; fast < nums.length; fast++) {
        // Compare with element 2 positions back
        if (nums[fast] !== nums[slow - 2]) {
            nums[slow] = nums[fast];
            slow++;
        }
    }
    
    return slow;
}

/**
 * Remove Duplicates - Alternative approach
 * Time: O(n), Space: O(1)
 */
function removeDuplicatesAlt(nums) {
    if (nums.length === 0) return 0;
    
    let uniqueCount = 1;  // First element is always unique
    
    for (let i = 1; i < nums.length; i++) {
        if (nums[i] !== nums[i - 1]) {
            nums[uniqueCount] = nums[i];
            uniqueCount++;
        }
    }
    
    return uniqueCount;
}

/**
 * Generic K duplicates allowed
 * Time: O(n), Space: O(1)
 */
function removeDuplicatesK(nums, k) {
    if (nums.length <= k) return nums.length;
    
    let slow = k;
    
    for (let fast = k; fast < nums.length; fast++) {
        // Compare with element k positions back
        if (nums[fast] !== nums[slow - k]) {
            nums[slow] = nums[fast];
            slow++;
        }
    }
    
    return slow;
}

// Test cases
function testRemoveDuplicates() {
    let nums1 = [1,1,2];
    console.log(removeDuplicates(nums1), nums1); // 2, [1,2,...]
    
    let nums2 = [0,0,1,1,1,2,2,3,3,4];
    console.log(removeDuplicates(nums2), nums2); // 5, [0,1,2,3,4,...]
    
    let nums3 = [1,1,1,2,2,3];
    console.log(removeDuplicatesII(nums3), nums3); // 5, [1,1,2,2,3,...]
    
    let nums4 = [1,2,3,4,5];
    console.log(removeDuplicates(nums4), nums4); // 5, [1,2,3,4,5]
}

testRemoveDuplicates();
```

## 🎨 The Approach

### Two-Pointer Strategy:
1. **Slow pointer** - Tracks position for next unique element
2. **Fast pointer** - Explores array for new unique values
3. **Compare and copy** - When unique found, copy to slow position
4. **Return length** - Slow pointer + 1 gives unique count

### Visual Walkthrough:
```
Initial: [0,0,1,1,1,2,2,3,3,4]
         s f
         
Step 1: nums[1]=0 == nums[0]=0, move fast
        [0,0,1,1,1,2,2,3,3,4]
         s   f
         
Step 2: nums[2]=1 != nums[0]=0, copy and move slow
        [0,1,1,1,1,2,2,3,3,4]
           s   f
           
Step 3: nums[3]=1 == nums[1]=1, move fast
        [0,1,1,1,1,2,2,3,3,4]
           s     f
           
Step 4: nums[4]=1 == nums[1]=1, move fast
        [0,1,1,1,1,2,2,3,3,4]
           s       f
           
Step 5: nums[5]=2 != nums[1]=1, copy and move slow
        [0,1,2,1,1,2,2,3,3,4]
             s       f

Continue...
Final:  [0,1,2,3,4,?,?,?,?,?]
                 ↑
              length=5
```

## 🤔 Why This Works

- **Sorted property**: Duplicates are adjacent
- **Two-pointer efficiency**: Single pass, no extra space
- **In-place modification**: Overwrite duplicates with unique values
- **Invariant maintained**: First `slow+1` elements are unique

## ⚠️ Common Pitfalls

1. **Unsorted array** - Algorithm assumes sorted input
2. **Off-by-one** - Return `slow + 1`, not `slow`
3. **Empty array** - Handle edge case
4. **Not modifying array** - Problem requires in-place modification

## 🎯 Variations

### Remove Duplicates II (Allow 2)
- Compare with element 2 positions back
- Start from index 2

### Allow K Duplicates
- Compare with element K positions back
- Generalized solution

### Remove All Duplicates
- Different problem: elements appearing more than once are completely removed

## 📊 Complexity Analysis

- **Time**: O(n) - Single pass through array
- **Space**: O(1) - Only using two pointers
- **Optimal**: Can't do better than O(n) for scanning

## 🔄 Related Problems

- **Remove Element** - Remove specific value
- **Move Zeroes** - Move specific elements to end
- **Remove Duplicates from Unsorted Array** - Requires different approach
- **Partition Array** - Rearrange based on condition
