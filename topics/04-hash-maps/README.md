# 🗺️ Hash Maps & Hash Tables

## Overview
Hash Maps (Hash Tables) provide O(1) average-case access time by using a hash function to map keys to array indices. They're essential for fast lookups, deduplication, and counting problems.

## Key Concepts

### Core Operations
| Operation | Average | Worst | Description |
|-----------|---------|-------|-------------|
| Insert | O(1) | O(n) | Add key-value pair |
| Delete | O(1) | O(n) | Remove key |
| Search | O(1) | O(n) | Find value by key |
| Update | O(1) | O(n) | Modify existing value |

### Collision Resolution
1. **Chaining** - Store collisions in linked lists
2. **Open Addressing** - Find next available slot
   - Linear Probing
   - Quadratic Probing
   - Double Hashing

### Load Factor
- Ratio of filled slots to total slots
- Typical threshold: 0.75
- Triggers resizing when exceeded

## Common Patterns

1. **Frequency Counting** - Count occurrences
2. **Two Sum Pattern** - Find pairs with target sum
3. **Deduplication** - Remove duplicates
4. **Grouping/Bucketing** - Group by common property
5. **Caching** - Store computed results
6. **Graph Representation** - Adjacency lists

## Problems in This Section

### Basic
- [Ice Cream Parlor](problems/ice-cream-parlor.md) - Two sum variant
- [Migratory Birds](problems/migratory-birds.md) - Frequency counting
- [Missing Numbers](problems/missing-numbers.md) - Find missing elements

### Intermediate
- [Colorful Number](problems/colorful-number.md) - Unique products
- [Group Anagrams](problems/group-anagrams.md) - String grouping
- [Longest Consecutive Sequence](problems/longest-consecutive-sequence.md) - Set operations

### Advanced
- [LFU Cache](problems/lfu-cache.md) - Complex cache implementation

## Implementation Examples

```javascript
// Basic HashMap usage in JavaScript
const map = new Map();

// Add/Update
map.set('key', 'value');

// Get
const value = map.get('key');

// Check existence
if (map.has('key')) { }

// Delete
map.delete('key');

// Iterate
for (const [key, value] of map) { }

// Size
console.log(map.size);
```

### Common Patterns

```javascript
// Frequency Counter
function countFrequencies(arr) {
    const freq = new Map();
    for (const item of arr) {
        freq.set(item, (freq.get(item) || 0) + 1);
    }
    return freq;
}

// Two Sum Pattern
function twoSum(nums, target) {
    const seen = new Map();
    for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];
        if (seen.has(complement)) {
            return [seen.get(complement), i];
        }
        seen.set(nums[i], i);
    }
    return [];
}

// Group By Pattern
function groupBy(items, keyFn) {
    const groups = new Map();
    for (const item of items) {
        const key = keyFn(item);
        if (!groups.has(key)) {
            groups.set(key, []);
        }
        groups.get(key).push(item);
    }
    return groups;
}
```

## JavaScript Specifics

### Map vs Object
```javascript
// Map - Any type as key, maintains insertion order
const map = new Map();
map.set(1, 'number key');
map.set('1', 'string key');
map.set({id: 1}, 'object key');

// Object - String/Symbol keys only
const obj = {};
obj['key'] = 'value';
obj[1] = 'converts to "1"';
```

### Set for Uniqueness
```javascript
// Remove duplicates
const unique = [...new Set(array)];

// Check membership
const set = new Set(array);
if (set.has(value)) { }

// Set operations
const intersection = new Set([...set1].filter(x => set2.has(x)));
const union = new Set([...set1, ...set2]);
```

## Advanced Techniques

### Rolling Hash
```javascript
// For substring matching
function rollingHash(str, windowSize) {
    const base = 256;
    const mod = 101;
    let hash = 0;
    let pow = 1;
    
    for (let i = 0; i < windowSize; i++) {
        hash = (hash * base + str.charCodeAt(i)) % mod;
        if (i < windowSize - 1) {
            pow = (pow * base) % mod;
        }
    }
    return {hash, pow};
}
```

### Consistent Hashing
```javascript
// For distributed systems
class ConsistentHash {
    constructor(nodes = []) {
        this.ring = new Map();
        this.sortedKeys = [];
        nodes.forEach(node => this.addNode(node));
    }
    
    hash(key) {
        // Simple hash function
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
        }
        return Math.abs(hash);
    }
}
```

## Tips for Success

1. **Choose the right structure** - Map vs Object vs Set
2. **Handle collisions** - Understand worst-case scenarios
3. **Consider space-time tradeoff** - Hash maps trade space for speed
4. **Key design** - Immutable keys, good hash distribution
5. **Initial capacity** - Prevent frequent resizing

## When to Use Hash Maps

✅ **Perfect for:**
- Fast lookups by key
- Counting/frequency problems
- Caching computed results
- Detecting duplicates
- Grouping related items
- Graph adjacency lists

❌ **Avoid when:**
- Need ordered traversal (use TreeMap/sorted array)
- Keys change frequently
- Memory is extremely limited
- Need range queries
