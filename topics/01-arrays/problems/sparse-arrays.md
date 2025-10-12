# Sparse Arrays

## 🎯 What Are We Trying To Solve?

Imagine you're analyzing search queries on a website. You have a list of all searches made, and you want to quickly answer: "How many times was this specific term searched?" This is a frequency counting problem - perfect for hash maps!

## 📝 The Problem

You have a collection of strings and a list of queries. For each query string, determine how many times it occurs in the collection of strings.

```
Example:
Strings: ["ab", "ab", "abc"]
Queries: ["ab", "abc", "bc"]

Results: [2, 1, 0]
- "ab" appears 2 times
- "abc" appears 1 time
- "bc" appears 0 times
```

## 📊 Examples

**Example 1:**
```
Input:
strings = ["aba", "baba", "aba", "xzxb"]
queries = ["aba", "xzxb", "ab"]

Output: [2, 1, 0]

Explanation:
- "aba" appears 2 times
- "xzxb" appears 1 time
- "ab" appears 0 times
```

**Example 2:**
```
Input:
strings = ["def", "de", "fgh"]
queries = ["de", "lmn", "fgh"]

Output: [1, 0, 1]
```

## 💻 The Code

```javascript
/**
 * Count occurrences of query strings in a collection
 * @param {string[]} strings - Collection of strings
 * @param {string[]} queries - Queries to search for
 * @returns {number[]} - Count of each query in the collection
 */
function sparseArrays(strings, queries) {
    // Step 1: Build frequency map
    const frequencyMap = new Map();
    
    for (const str of strings) {
        // If string exists, increment count; otherwise set to 1
        frequencyMap.set(str, (frequencyMap.get(str) || 0) + 1);
    }
    
    // Step 2: Process queries
    const results = [];
    
    for (const query of queries) {
        // Get count from map, default to 0 if not found
        results.push(frequencyMap.get(query) || 0);
    }
    
    return results;
}

// Alternative: Using object instead of Map
function sparseArraysObject(strings, queries) {
    // Build frequency object
    const frequency = {};
    
    for (const str of strings) {
        frequency[str] = (frequency[str] || 0) + 1;
    }
    
    // Process queries
    return queries.map(query => frequency[query] || 0);
}

// Alternative: One-liner functional approach
function sparseArraysFunctional(strings, queries) {
    return queries.map(query => 
        strings.filter(str => str === query).length
    );
}

// Alternative: Using reduce for frequency map
function sparseArraysReduce(strings, queries) {
    const frequency = strings.reduce((acc, str) => {
        acc[str] = (acc[str] || 0) + 1;
        return acc;
    }, {});
    
    return queries.map(q => frequency[q] || 0);
}

// Test cases
console.log(sparseArrays(
    ["aba", "baba", "aba", "xzxb"],
    ["aba", "xzxb", "ab"]
));  // [2, 1, 0]

console.log(sparseArrays(
    ["def", "de", "fgh"],
    ["de", "lmn", "fgh"]
));  // [1, 0, 1]

console.log(sparseArrays([], ["test"]));  // [0]
console.log(sparseArrays(["a", "a", "a"], ["a", "b"]));  // [3, 0]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `sparseArrays(["aba", "baba", "aba", "xzxb"], ["aba", "xzxb", "ab"])`:

```
Step 1: Build frequency map
Processing strings:
- "aba": map = {"aba": 1}
- "baba": map = {"aba": 1, "baba": 1}
- "aba": map = {"aba": 2, "baba": 1}
- "xzxb": map = {"aba": 2, "baba": 1, "xzxb": 1}

Step 2: Process queries
- Query "aba": map.get("aba") = 2 → results = [2]
- Query "xzxb": map.get("xzxb") = 1 → results = [2, 1]
- Query "ab": map.get("ab") = undefined → 0 → results = [2, 1, 0]

Return: [2, 1, 0]
```

## 📊 Visual Understanding

```
Strings Array:        Frequency Map:
["aba",               {
 "baba",        →      "aba": 2,
 "aba",                "baba": 1,
 "xzxb"]               "xzxb": 1
                      }

Queries:              Results:
["aba",         →     [2,      (found in map)
 "xzxb",        →      1,      (found in map)
 "ab"]          →      0]      (not in map)
```

## ⏱️ Complexity Analysis

**Hash Map Approach:**
- Time: O(n + m) where n = strings length, m = queries length
  - O(n) to build frequency map
  - O(m) to process queries (O(1) per lookup)
- Space: O(n) for the frequency map

**Filter Approach (Functional):**
- Time: O(n × m) - for each query, scan all strings
- Space: O(1) - no extra storage

The hash map approach is much faster for large datasets!

## 🚫 Common Mistakes

1. **Not handling missing keys**
   ```javascript
   // Wrong: Will return undefined for missing keys
   results.push(frequencyMap.get(query));
   
   // Right: Use default value
   results.push(frequencyMap.get(query) || 0);
   ```

2. **Case sensitivity issues**
   ```javascript
   // If case-insensitive matching is needed:
   function sparseArraysCaseInsensitive(strings, queries) {
       const frequency = {};
       
       // Store lowercase versions
       for (const str of strings) {
           const lower = str.toLowerCase();
           frequency[lower] = (frequency[lower] || 0) + 1;
       }
       
       // Query with lowercase
       return queries.map(q => frequency[q.toLowerCase()] || 0);
   }
   ```

3. **Mutating input arrays**
   ```javascript
   // Wrong: Sorting mutates original
   strings.sort();
   
   // Right: Work with copies if needed
   const sortedStrings = [...strings].sort();
   ```

## 🔄 Variations

```javascript
// Return indices where queries appear
function sparseArrayIndices(strings, queries) {
    const indexMap = {};
    
    strings.forEach((str, idx) => {
        if (!indexMap[str]) indexMap[str] = [];
        indexMap[str].push(idx);
    });
    
    return queries.map(q => indexMap[q] || []);
}

// Count with wildcards (simple pattern matching)
function sparseArraysWithWildcard(strings, queries) {
    return queries.map(query => {
        const regex = new RegExp('^' + query.replace('*', '.*') + '$');
        return strings.filter(str => regex.test(str)).length;
    });
}

// Get most frequent strings
function getMostFrequent(strings, k) {
    const frequency = {};
    
    for (const str of strings) {
        frequency[str] = (frequency[str] || 0) + 1;
    }
    
    return Object.entries(frequency)
        .sort((a, b) => b[1] - a[1])
        .slice(0, k)
        .map(([str, count]) => ({ string: str, count }));
}
```

## 🔍 Map vs Object

```javascript
// Using Map (recommended for this problem)
const map = new Map();
map.set("key", 1);           // Set value
map.get("key");               // Get value
map.has("key");               // Check existence
map.delete("key");            // Remove key
map.size;                     // Number of entries

// Using Object
const obj = {};
obj["key"] = 1;               // Set value
obj["key"];                   // Get value
"key" in obj;                 // Check existence
delete obj["key"];            // Remove key
Object.keys(obj).length;      // Number of entries

// Map advantages:
// - Any type can be a key (not just strings)
// - Better performance for frequent additions/deletions
// - Maintains insertion order
// - Has size property
```

## 🔗 Related Problems

- **Word Frequency** - Count word occurrences in text
- **First Unique Character** - Find first non-repeating character
- **Group Anagrams** - Group strings by frequency pattern
- **Top K Frequent Elements** - Find most common elements

## 💡 Key Takeaways

1. **Hash Maps for Counting**: Perfect data structure for frequency problems
2. **Time-Space Tradeoff**: Hash map uses more space but gives O(1) lookups
3. **Default Values**: Always handle missing keys gracefully
4. **Map vs Object**: Choose based on your needs (Map is often better)

This problem is a fundamental pattern that appears in many real-world scenarios!
