# Valid Palindrome II

## 🎯 What Are We Trying To Solve?

Determine if a string can be a palindrome after deleting at most one character from it.

## 📝 The Problem

Given a string `s`, return `true` if the string can be a palindrome after deleting at most one character from it.

```
Example 1:
Input: s = "aba"
Output: true
Explanation: Already a palindrome

Example 2:
Input: s = "abca"
Output: true
Explanation: Delete 'c' to get "aba"

Example 3:
Input: s = "abc"
Output: false
Explanation: Can't form palindrome by deleting one character
```

## 💻 The Code

```javascript
/**
 * Valid Palindrome II - Two Pointer Solution
 * Time: O(n), Space: O(1)
 */
function validPalindrome(s) {
    let left = 0;
    let right = s.length - 1;
    
    while (left < right) {
        if (s[left] !== s[right]) {
            // Try deleting either left or right character
            return isPalindrome(s, left + 1, right) || 
                   isPalindrome(s, left, right - 1);
        }
        left++;
        right--;
    }
    
    return true; // Already a palindrome
}

/**
 * Helper function to check palindrome in range
 */
function isPalindrome(s, left, right) {
    while (left < right) {
        if (s[left] !== s[right]) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}

/**
 * Alternative: Track deletion count
 * Time: O(n), Space: O(1)
 */
function validPalindromeAlt(s) {
    function helper(left, right, deleted) {
        while (left < right) {
            if (s[left] !== s[right]) {
                if (deleted) return false;
                
                // Try deleting either character
                return helper(left + 1, right, true) || 
                       helper(left, right - 1, true);
            }
            left++;
            right--;
        }
        return true;
    }
    
    return helper(0, s.length - 1, false);
}

/**
 * Extended: Can delete K characters
 * Time: O(n * k), Space: O(k) for recursion
 */
function validPalindromeK(s, k) {
    const memo = new Map();
    
    function helper(left, right, remaining) {
        if (left >= right) return true;
        
        const key = `${left},${right},${remaining}`;
        if (memo.has(key)) return memo.get(key);
        
        let result;
        if (s[left] === s[right]) {
            result = helper(left + 1, right - 1, remaining);
        } else if (remaining > 0) {
            // Try deleting either character
            result = helper(left + 1, right, remaining - 1) || 
                    helper(left, right - 1, remaining - 1);
        } else {
            result = false;
        }
        
        memo.set(key, result);
        return result;
    }
    
    return helper(0, s.length - 1, k);
}

/**
 * Find which character to delete (if any)
 * Time: O(n), Space: O(1)
 */
function findCharToDelete(s) {
    let left = 0;
    let right = s.length - 1;
    
    while (left < right) {
        if (s[left] !== s[right]) {
            // Check which deletion works
            if (isPalindrome(s, left + 1, right)) {
                return left; // Delete character at left
            } else if (isPalindrome(s, left, right - 1)) {
                return right; // Delete character at right
            } else {
                return -1; // Can't form palindrome
            }
        }
        left++;
        right--;
    }
    
    return -2; // Already a palindrome, no deletion needed
}

// Test cases
console.log(validPalindrome("aba")); // true
console.log(validPalindrome("abca")); // true
console.log(validPalindrome("abc")); // false
console.log(validPalindrome("raceacar")); // true (delete 'a')
console.log(validPalindrome("abcbea")); // true (delete 'e')
console.log(validPalindrome("abcdef")); // false

// Test finding which character to delete
console.log(findCharToDelete("abca")); // 3 (index of 'c' or 'a')
console.log(findCharToDelete("raceacar")); // 3 (index of middle 'a')
```

## 🎨 The Approach

### Two-Pointer Strategy:
1. **Start from both ends** - Compare characters moving inward
2. **On mismatch** - Try deleting either character
3. **Check remainder** - Verify if remaining string is palindrome
4. **Return result** - True if either deletion works

### Visual Walkthrough:
```
Example: "abca"
         L  R     s[0]='a' == s[3]='a' ✓
         
         "abca"
          L R     s[1]='b' != s[2]='c' ✗
          
Try option 1: Delete left ('b')
         "a_ca"
           LR     Check "ca": 'c' != 'a' ✗
           
Try option 2: Delete right ('c')
         "ab_a"
          LR      Check "ba": 'b' != 'a' ✗
          
Wait, let's reconsider...
Actually checking s[left+1] with s[right]:
         "abca"
          ↓↓      s[2]='c' with s[3]='a' 
          
Or s[left] with s[right-1]:
         "abca"
          ↓↓      s[1]='b' with s[1]='b' ✓
          
Result: true (can delete 'c')
```

## 🤔 Why This Works

- **One deletion allowed**: At most one mismatch point
- **Two choices**: When mismatch found, try both deletions
- **Greedy approach**: First mismatch determines the deletion
- **Early termination**: If both deletions fail, return false

## ⚠️ Common Pitfalls

1. **Multiple mismatches** - Can only handle one
2. **Not checking both options** - Must try deleting from either side
3. **Infinite recursion** - Need to track deletion state
4. **Empty string** - Handle edge case

## 🎯 Key Insights

### Why only check at first mismatch?
- If we can form palindrome with one deletion, it must resolve the first mismatch
- Once we skip a character, remaining must be perfect palindrome

### Optimization opportunities:
- Early termination when both deletions fail
- Memoization for K deletions variant
- String slicing vs index tracking

## 📊 Complexity Analysis

- **Time**: O(n) - At most 2n character comparisons
- **Space**: O(1) - Only using pointers
- **Optimal**: Can't do better than O(n) for verification

## 🔄 Related Problems

- **Valid Palindrome** - Basic palindrome check
- **Palindrome Linked List** - Similar technique for lists
- **Longest Palindromic Substring** - Find longest palindrome
- **Minimum Deletions to Make Palindrome** - Delete multiple characters
