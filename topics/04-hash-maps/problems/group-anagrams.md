# Group Anagrams

## 🎯 What Are We Trying To Solve?

Imagine you're organizing a word game night and need to group scrambled words that are made from the same letters. Words like "eat", "tea", and "ate" all use the same letters (just rearranged), so they belong in the same group. That's exactly what we're doing here - grouping anagrams together!

## 📝 The Problem

Given an array of strings, group the anagrams together. An anagram is a word formed by rearranging the letters of another word, using all the original letters exactly once.

```
Input: ["eat", "tea", "tan", "ate", "nat", "bat"]
Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]

The groups can be in any order, and words within groups can be in any order.
```

## 🧠 How to Think About This

The key insight: Anagrams have the same letters, just in different order. If we sort the letters of each word, anagrams will become identical!

### Visual Process
```
"eat" → sort → "aet"
"tea" → sort → "aet"  } Same sorted form = anagrams!
"ate" → sort → "aet"

"tan" → sort → "ant"
"nat" → sort → "ant"  } Same sorted form = anagrams!

"bat" → sort → "abt"  } Unique sorted form = own group
```

## 💻 The Code

### Approach 1: Sort Characters (Intuitive)

```javascript
/**
 * Group anagrams using sorted strings as keys
 * Time: O(n * k log k) where n = number of strings, k = max string length
 * Space: O(n * k) for storing all strings in the map
 */
function groupAnagrams(strs) {
    // Map to store: sorted_string → [original_strings]
    const map = new Map();
    
    for (const str of strs) {
        // Sort the characters to create a key
        // All anagrams will have the same sorted form
        const key = str.split('').sort().join('');
        
        // If this sorted form hasn't been seen, create new group
        if (!map.has(key)) {
            map.set(key, []);
        }
        
        // Add the original string to its anagram group
        map.get(key).push(str);
    }
    
    // Convert map values to array
    // Each value is an array of anagrams
    return Array.from(map.values());
}

// Test with examples
console.log(groupAnagrams(["eat", "tea", "tan", "ate", "nat", "bat"]));
// Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]

console.log(groupAnagrams([""]));
// Output: [[""]]

console.log(groupAnagrams(["a"]));
// Output: [["a"]]
```

### Approach 2: Character Count (Optimal)

```javascript
/**
 * Group anagrams using character frequency
 * Time: O(n * k) where n = number of strings, k = max string length
 * Space: O(n * k)
 */
function groupAnagramsOptimal(strs) {
    const map = new Map();
    
    for (const str of strs) {
        // Count frequency of each character
        // Array of 26 zeros (one for each letter a-z)
        const count = new Array(26).fill(0);
        
        // Count occurrences of each character
        for (const char of str) {
            // 'a' → 0, 'b' → 1, ..., 'z' → 25
            const index = char.charCodeAt(0) - 97;
            count[index]++;
        }
        
        // Create unique key from character counts
        // Example: "aab" → "2,1,0,0,0..." (2 a's, 1 b, 0 c's...)
        const key = count.join('#');
        
        if (!map.has(key)) {
            map.set(key, []);
        }
        
        map.get(key).push(str);
    }
    
    return Array.from(map.values());
}
```

### Approach 3: Prime Number Product (Mathematical)

```javascript
/**
 * Group anagrams using prime number products
 * Each letter maps to a unique prime, anagrams have same product
 * Note: May overflow for very long strings
 */
function groupAnagramsPrime(strs) {
    // Map each letter to a unique prime number
    const primes = [2,3,5,7,11,13,17,19,23,29,31,37,41,
                   43,47,53,59,61,67,71,73,79,83,89,97,101];
    
    const map = new Map();
    
    for (const str of strs) {
        let key = 1;
        
        // Multiply primes for each character
        // Anagrams will have the same product!
        for (const char of str) {
            const index = char.charCodeAt(0) - 97;
            key *= primes[index];
        }
        
        if (!map.has(key)) {
            map.set(key, []);
        }
        
        map.get(key).push(str);
    }
    
    return Array.from(map.values());
}
```

## 🎨 Step-by-Step Walkthrough

Let's trace through with ["eat", "tea", "tan", "ate", "nat", "bat"]:

```
Step 1: Process "eat"
- Sort: "aet"
- Map: {"aet": ["eat"]}

Step 2: Process "tea"
- Sort: "aet" (same as "eat"!)
- Map: {"aet": ["eat", "tea"]}

Step 3: Process "tan"
- Sort: "ant"
- Map: {"aet": ["eat", "tea"], "ant": ["tan"]}

Step 4: Process "ate"
- Sort: "aet" (matches existing key!)
- Map: {"aet": ["eat", "tea", "ate"], "ant": ["tan"]}

Step 5: Process "nat"
- Sort: "ant" (matches "tan"'s key!)
- Map: {"aet": ["eat", "tea", "ate"], "ant": ["tan", "nat"]}

Step 6: Process "bat"
- Sort: "abt"
- Map: {"aet": ["eat", "tea", "ate"], "ant": ["tan", "nat"], "abt": ["bat"]}

Final: Convert map values to array
Result: [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
```

## 📊 Complexity Comparison

| Approach | Time Complexity | Space Complexity | Notes |
|----------|----------------|------------------|-------|
| Sort Characters | O(n * k log k) | O(n * k) | Simple, but sorting overhead |
| Character Count | O(n * k) | O(n * k) | Optimal for lowercase letters |
| Prime Product | O(n * k) | O(n * k) | Risk of overflow |

Where:
- n = number of strings
- k = maximum length of a string

## 🚫 Common Mistakes

1. **Not Handling Empty Strings**
   ```javascript
   // Wrong: Crashes on empty string
   const key = str.split('').sort().join('');
   
   // Right: Works with empty strings
   if (str === "") {
       // Handle separately or let sort work naturally
   }
   ```

2. **Using Objects Instead of Map**
   ```javascript
   // Problematic: Object keys are always strings
   const groups = {};
   groups[key] = groups[key] || [];
   
   // Better: Map handles any key type
   const groups = new Map();
   ```

3. **Not Preserving Original Strings**
   ```javascript
   // Wrong: Only storing sorted versions
   map.set(sortedStr, sortedStr);
   
   // Right: Store original strings
   map.get(sortedStr).push(originalStr);
   ```

## 🔄 Variations

### Group by Pattern
```javascript
// Group strings by character pattern
// "abb" and "egg" have same pattern: one unique, two repeated
function groupByPattern(strs) {
    const map = new Map();
    
    for (const str of strs) {
        const pattern = [];
        const charMap = new Map();
        let id = 0;
        
        for (const char of str) {
            if (!charMap.has(char)) {
                charMap.set(char, id++);
            }
            pattern.push(charMap.get(char));
        }
        
        const key = pattern.join(',');
        
        if (!map.has(key)) {
            map.set(key, []);
        }
        map.get(key).push(str);
    }
    
    return Array.from(map.values());
}

// Test: ["abb", "egg", "foo", "ddd"]
// Result: [["abb", "egg", "foo"], ["ddd"]]
```

## 🔗 Related Problems

- **Valid Anagram** - Check if two strings are anagrams
- **Find All Anagrams in String** - Sliding window anagram search
- **Group Shifted Strings** - Group by shifting pattern
- **Palindrome Pairs** - Find pairs that form palindromes

## 💡 Key Takeaways

1. **Hash Maps are Perfect for Grouping**: Use unique keys to group similar items
2. **Sorting Creates Canonical Form**: Anagrams become identical when sorted
3. **Character Counting is Optimal**: O(n) is better than O(n log n) sorting
4. **Multiple Valid Approaches**: Choose based on constraints and clarity

This pattern of creating a "signature" or "key" for grouping appears in many problems!
