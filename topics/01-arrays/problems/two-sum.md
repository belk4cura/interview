# Two Sum

## 🎯 What Are We Trying To Solve?

Imagine you're at a store with a gift card for exactly $9. You want to buy exactly two items that together cost $9. You see items priced at $2, $7, $11, and $15. Which two items should you buy?

The answer is the $2 item and the $7 item because 2 + 7 = 9!

This is the Two Sum problem: Given a list of numbers and a target sum, find which two numbers add up to that target.

## 📝 Examples

**Example 1:**
```
Input: nums = [2, 7, 11, 15], target = 9
Output: [0, 1]
Why? nums[0] + nums[1] = 2 + 7 = 9
```

**Example 2:**
```
Input: nums = [3, 2, 4], target = 6
Output: [1, 2]
Why? nums[1] + nums[2] = 2 + 4 = 6
```

**Example 3:**
```
Input: nums = [3, 3], target = 6
Output: [0, 1]
Why? nums[0] + nums[1] = 3 + 3 = 6
```

## 🧠 How Do We Think About This?

### Approach 1: The Brute Force Way (Check Every Pair)
1. Pick the first number
2. Try adding it with every other number
3. If we find a match, we're done!
4. If not, pick the next number and repeat

This works but is slow for large lists (O(n²) time).

### Approach 2: The Smart Way (Using a Hash Map)
Think of it like this:
1. As we go through the list, we remember each number we've seen
2. For each new number, we ask: "What number would I need to reach the target?"
3. We check if we've already seen that number
4. If yes, we found our pair! If no, remember this number and continue

## 💻 The Code

```javascript
function twoSum(nums, target) {
    // This is like our memory - we'll remember numbers we've seen
    // Map is like a phonebook: number -> its position in the array
    const seen = new Map();
    
    // Go through each number in the array
    for (let i = 0; i < nums.length; i++) {
        // What number would we need to add to nums[i] to get target?
        const complement = target - nums[i];
        
        // Have we seen this complement before?
        if (seen.has(complement)) {
            // Yes! We found our pair
            // Return the positions of both numbers
            return [seen.get(complement), i];
        }
        
        // Haven't found a match yet
        // Remember this number and where we saw it
        seen.set(nums[i], i);
    }
    
    // If we get here, no two numbers add up to target
    return [];
}

// Test the function
console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
console.log(twoSum([3, 2, 4], 6));      // [1, 2]
console.log(twoSum([3, 3], 6));         // [0, 1]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `twoSum([2, 7, 11, 15], 9)`:

```
Step 1: i = 0, nums[i] = 2
  - complement = 9 - 2 = 7
  - Have we seen 7? No
  - Remember: 2 is at position 0
  - seen = {2 → 0}

Step 2: i = 1, nums[i] = 7
  - complement = 9 - 7 = 2
  - Have we seen 2? Yes! At position 0
  - Return [0, 1] ✓
```

## ⏱️ How Fast Is This?

- **Time Complexity:** O(n) - We look at each number once
  - In plain English: If we have 1000 numbers, we do about 1000 operations
  
- **Space Complexity:** O(n) - We might store all numbers in our Map
  - In plain English: If we have 1000 numbers, we might need memory for 1000 entries

## 🚫 Common Mistakes

1. **Forgetting that we need indices, not values**
   ```javascript
   // Wrong: Returns the values
   return [complement, nums[i]];
   
   // Right: Returns the positions
   return [seen.get(complement), i];
   ```

2. **Using the same element twice**
   ```javascript
   // This is why we check seen BEFORE adding current number
   // Otherwise [3, 2, 4] with target 6 might incorrectly use 3 twice
   ```

3. **Not handling "no solution" case**
   - Always return empty array if no pair is found

## 🔄 Alternative Solutions

### Brute Force (Simpler but Slower)
```javascript
function twoSumBruteForce(nums, target) {
    // Check every possible pair
    for (let i = 0; i < nums.length; i++) {
        for (let j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] === target) {
                return [i, j];
            }
        }
    }
    return [];
}
```
- Time: O(n²) - Much slower for large arrays
- Space: O(1) - Uses no extra memory

### Using Sorting (If we didn't need indices)
```javascript
function twoSumValues(nums, target) {
    // Sort the array first
    nums.sort((a, b) => a - b);
    
    let left = 0;
    let right = nums.length - 1;
    
    while (left < right) {
        const sum = nums[left] + nums[right];
        if (sum === target) {
            return [nums[left], nums[right]]; // Returns values, not indices
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    return [];
}
```
- Time: O(n log n) - Due to sorting
- Space: O(1) - No extra space needed

## 🔗 Related Problems

- **Three Sum**: Find three numbers that sum to target
- **Two Sum II**: When the array is already sorted
- **Two Sum IV**: When working with a Binary Search Tree
- **Subarray Sum Equals K**: Find subarrays with given sum

## 💡 Key Takeaway

The hash map approach trades space for speed. By remembering what we've seen, we can find pairs in one pass instead of checking every combination. This pattern appears in many interview problems!
