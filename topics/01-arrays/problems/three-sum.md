# Three Sum

## 🎯 What Are We Trying To Solve?

Find all unique triplets in an array that sum to zero. This is a classic two-pointer problem that extends the Two Sum concept.

## 📝 The Problem

Given an integer array, return all unique triplets [nums[i], nums[j], nums[k]] such that i != j, i != k, j != k, and nums[i] + nums[j] + nums[k] == 0.

```
Example 1:
Input: nums = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]

Example 2:
Input: nums = [0, 1, 1]
Output: []

Example 3:
Input: nums = [0, 0, 0]
Output: [[0, 0, 0]]
```

## 💻 The Code

```javascript
/**
 * Three Sum - Two Pointer Solution
 * Time: O(n²), Space: O(1) excluding output
 */
function threeSum(nums) {
    const result = [];
    
    // Sort array first for two-pointer technique
    nums.sort((a, b) => a - b);
    
    // Fix first element and find pairs
    for (let i = 0; i < nums.length - 2; i++) {
        // Skip duplicates for first element
        if (i > 0 && nums[i] === nums[i - 1]) continue;
        
        // Two pointers for remaining array
        let left = i + 1;
        let right = nums.length - 1;
        const target = -nums[i]; // We need a + b = -c
        
        while (left < right) {
            const sum = nums[left] + nums[right];
            
            if (sum === target) {
                // Found a triplet
                result.push([nums[i], nums[left], nums[right]]);
                
                // Skip duplicates for second element
                while (left < right && nums[left] === nums[left + 1]) left++;
                // Skip duplicates for third element
                while (left < right && nums[right] === nums[right - 1]) right--;
                
                left++;
                right--;
            } else if (sum < target) {
                // Need larger sum
                left++;
            } else {
                // Need smaller sum
                right--;
            }
        }
    }
    
    return result;
}

/**
 * Alternative: Using HashSet (less efficient)
 * Time: O(n²), Space: O(n)
 */
function threeSumHashSet(nums) {
    nums.sort((a, b) => a - b);
    const result = [];
    
    for (let i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] === nums[i - 1]) continue;
        
        const seen = new Set();
        const target = -nums[i];
        
        for (let j = i + 1; j < nums.length; j++) {
            const complement = target - nums[j];
            
            if (seen.has(complement)) {
                result.push([nums[i], complement, nums[j]]);
                // Skip duplicates
                while (j + 1 < nums.length && nums[j] === nums[j + 1]) j++;
            }
            seen.add(nums[j]);
        }
    }
    
    return result;
}

// Test cases
console.log(threeSum([-1, 0, 1, 2, -1, -4])); // [[-1,-1,2],[-1,0,1]]
console.log(threeSum([0, 1, 1])); // []
console.log(threeSum([0, 0, 0])); // [[0,0,0]]
console.log(threeSum([-2, 0, 1, 1, 2])); // [[-2,0,2],[-2,1,1]]
```

## 🎨 The Approach

### Two-Pointer Strategy:
1. **Sort the array** - Essential for two-pointer technique
2. **Fix first element** - Iterate through array
3. **Find pairs** - Use two pointers to find complementary pairs
4. **Skip duplicates** - Avoid duplicate triplets

### Visual Walkthrough:
```
Array: [-1, 0, 1, 2, -1, -4]
After sorting: [-4, -1, -1, 0, 1, 2]

Step 1: Fix -4, find pairs that sum to 4
        [-4, -1, -1, 0, 1, 2]
          ^   L           R
        No valid pairs

Step 2: Fix -1, find pairs that sum to 1
        [-4, -1, -1, 0, 1, 2]
              ^   L        R
        Found: -1 + 0 + 1 = 0 ✓
        Found: -1 + (-1) + 2 = 0 ✓

Step 3: Skip duplicate -1
Step 4: Fix 0, find pairs that sum to 0
        [-4, -1, -1, 0, 1, 2]
                     ^  L  R
        1 + 2 = 3 > 0, move R left
        No valid pairs
```

## 🤔 Why This Works

- **Sorting enables two-pointer**: Once sorted, we can move pointers based on sum comparison
- **Fixing one element**: Reduces problem to Two Sum
- **Duplicate handling**: Sorting groups duplicates together for easy skipping

## ⚠️ Common Pitfalls

1. **Forgetting to sort** - Two-pointer won't work on unsorted array
2. **Not handling duplicates** - Results in duplicate triplets
3. **Index boundaries** - Ensure i < length - 2 for triplets
4. **Integer overflow** - Be careful with large numbers

## 🎯 Variations

- **3Sum Closest**: Find triplet with sum closest to target
- **3Sum Smaller**: Count triplets with sum less than target
- **4Sum**: Extend to quadruplets
- **K-Sum**: Generalize to K elements

## 📊 Complexity Analysis

- **Time**: O(n²) - O(n log n) for sorting + O(n²) for nested loops
- **Space**: O(1) or O(n) depending on sorting algorithm
- **Result space**: O(n²) in worst case for output
