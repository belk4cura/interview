# Longest Consecutive Sequence

## 🎯 What Are We Trying To Solve?

Finding the longest consecutive sequence is like finding the longest straight in a shuffled deck of cards - we need to identify which numbers form unbroken chains.

## 📝 The Problem

Given an unsorted array of integers, find the length of the longest consecutive elements sequence in O(n) time.

```
Example:
Input: [100, 4, 200, 1, 3, 2]
Output: 4
Explanation: [1, 2, 3, 4]

Input: [0,3,7,2,5,8,4,6,0,1]
Output: 9
```

## 💻 The Code

```javascript
/**
 * Hash Set Solution
 * Time: O(n), Space: O(n)
 */
function longestConsecutive(nums) {
    const numSet = new Set(nums);
    let maxLength = 0;
    
    for (const num of numSet) {
        // Only start counting from beginning of sequence
        if (!numSet.has(num - 1)) {
            let currentNum = num;
            let currentLength = 1;
            
            // Count consecutive numbers
            while (numSet.has(currentNum + 1)) {
                currentNum++;
                currentLength++;
            }
            
            maxLength = Math.max(maxLength, currentLength);
        }
    }
    
    return maxLength;
}

/**
 * Union-Find Solution
 */
function longestConsecutiveUF(nums) {
    if (nums.length === 0) return 0;
    
    const parent = new Map();
    const size = new Map();
    
    function find(x) {
        if (!parent.has(x)) {
            parent.set(x, x);
            size.set(x, 1);
        }
        if (parent.get(x) !== x) {
            parent.set(x, find(parent.get(x)));
        }
        return parent.get(x);
    }
    
    function union(x, y) {
        const rootX = find(x);
        const rootY = find(y);
        
        if (rootX !== rootY) {
            parent.set(rootX, rootY);
            size.set(rootY, size.get(rootX) + size.get(rootY));
        }
    }
    
    for (const num of nums) {
        find(num);
        if (parent.has(num - 1)) union(num, num - 1);
        if (parent.has(num + 1)) union(num, num + 1);
    }
    
    return Math.max(...size.values());
}
```

## 🎨 Key Insights

- **Set for O(1) Lookup**: Check existence instantly
- **Start of Sequence**: Only count from first element
- **Linear Scan**: Each element visited at most twice

## ⏱️ Complexity

- Time: O(n) - Each element processed once
- Space: O(n) - Hash set storage
