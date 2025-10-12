# LFU Cache (Least Frequently Used)

## 🎯 What Are We Trying To Solve?

Imagine you're managing a small bookshelf with limited space. When it's full and you need to add a new book, you remove the book that's been read the least. If multiple books have been read the same (minimal) number of times, you remove the one that was read least recently. That's an LFU cache - it tracks both frequency and recency!

## 📝 The Problem

Design and implement a data structure for a Least Frequently Used (LFU) cache with these operations:
- `get(key)`: Get the value and increase its use count
- `put(key, value)`: Set or insert the value
- When capacity is reached, remove the least frequently used item
- If there's a tie in frequency, remove the least recently used among them

All operations must run in O(1) time complexity!

## 🧠 How to Think About This

We need to track THREE things:
1. **Key → Value** mapping (the actual cache)
2. **Key → Frequency** mapping (how often used)
3. **Frequency → Keys** mapping (which keys have each frequency)

### Visual Structure
```
Frequency Buckets:
Freq 1: [key3 → key5 → key2]  (least recent → most recent)
Freq 2: [key1 → key4]
Freq 3: [key6]

When evicting: Remove key3 (leftmost in lowest frequency)
```

## 💻 The Code

```javascript
/**
 * LFU Cache with O(1) operations
 */
class LFUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.minFreq = 0;
        
        // Three hash maps for O(1) access
        this.keyToVal = new Map();    // key → value
        this.keyToFreq = new Map();   // key → frequency
        this.freqToKeys = new Map();  // frequency → Set of keys
    }
    
    /**
     * Get value and increase frequency
     * Time: O(1)
     */
    get(key) {
        // Key doesn't exist
        if (!this.keyToVal.has(key)) {
            return -1;
        }
        
        // Increase frequency since we're accessing it
        this.increaseFreq(key);
        
        return this.keyToVal.get(key);
    }
    
    /**
     * Add/update key-value pair
     * Time: O(1)
     */
    put(key, value) {
        // Edge case: zero capacity
        if (this.capacity === 0) return;
        
        // Case 1: Key exists - update value and frequency
        if (this.keyToVal.has(key)) {
            this.keyToVal.set(key, value);
            this.increaseFreq(key);
            return;
        }
        
        // Case 2: New key - check capacity
        if (this.keyToVal.size >= this.capacity) {
            this.evict();
        }
        
        // Add new key with frequency 1
        this.keyToVal.set(key, value);
        this.keyToFreq.set(key, 1);
        
        // Add to frequency 1 bucket
        if (!this.freqToKeys.has(1)) {
            this.freqToKeys.set(1, new Set());
        }
        this.freqToKeys.get(1).add(key);
        
        // Update minimum frequency
        this.minFreq = 1;
    }
    
    /**
     * Increase frequency of a key
     * Time: O(1)
     */
    increaseFreq(key) {
        const freq = this.keyToFreq.get(key);
        const newFreq = freq + 1;
        
        // Update frequency mapping
        this.keyToFreq.set(key, newFreq);
        
        // Remove from old frequency bucket
        const oldFreqKeys = this.freqToKeys.get(freq);
        oldFreqKeys.delete(key);
        
        // If old frequency bucket is empty, remove it
        if (oldFreqKeys.size === 0) {
            this.freqToKeys.delete(freq);
            
            // If this was the minimum frequency, increment it
            if (this.minFreq === freq) {
                this.minFreq++;
            }
        }
        
        // Add to new frequency bucket
        if (!this.freqToKeys.has(newFreq)) {
            this.freqToKeys.set(newFreq, new Set());
        }
        this.freqToKeys.get(newFreq).add(key);
    }
    
    /**
     * Evict least frequently used item
     * Time: O(1)
     */
    evict() {
        // Get keys with minimum frequency
        const minFreqKeys = this.freqToKeys.get(this.minFreq);
        
        // Get the least recently used (first in set)
        // Sets maintain insertion order in JavaScript!
        const keyToEvict = minFreqKeys.values().next().value;
        
        // Remove from all maps
        minFreqKeys.delete(keyToEvict);
        
        // Clean up empty frequency bucket
        if (minFreqKeys.size === 0) {
            this.freqToKeys.delete(this.minFreq);
        }
        
        this.keyToVal.delete(keyToEvict);
        this.keyToFreq.delete(keyToEvict);
    }
}

// Test the LFU Cache
const cache = new LFUCache(2);

cache.put(1, 1);        // cache: {1=1}, freq: {1=[1]}
cache.put(2, 2);        // cache: {1=1, 2=2}, freq: {1=[1,2]}
console.log(cache.get(1)); // returns 1, freq: {1=[2], 2=[1]}
cache.put(3, 3);        // evicts key 2, cache: {1=1, 3=3}
console.log(cache.get(2)); // returns -1 (not found)
console.log(cache.get(3)); // returns 3
cache.put(4, 4);        // evicts key 1, cache: {3=3, 4=4}
console.log(cache.get(1)); // returns -1 (not found)
console.log(cache.get(3)); // returns 3
console.log(cache.get(4)); // returns 4
```

## 🎨 Step-by-Step Walkthrough

Let's trace through the example:

```
Initial: capacity = 2, cache is empty

put(1, 1):
- Add key 1 with value 1
- keyToVal: {1→1}
- keyToFreq: {1→1}
- freqToKeys: {1→{1}}
- minFreq: 1

put(2, 2):
- Add key 2 with value 2
- keyToVal: {1→1, 2→2}
- keyToFreq: {1→1, 2→1}
- freqToKeys: {1→{1,2}}
- minFreq: 1

get(1):
- Access key 1, increase frequency
- keyToVal: {1→1, 2→2}
- keyToFreq: {1→2, 2→1}
- freqToKeys: {1→{2}, 2→{1}}
- minFreq: 1
- Returns: 1

put(3, 3):
- Cache full! Evict least frequent (key 2)
- Remove key 2 (freq=1, only one with minFreq)
- Add key 3
- keyToVal: {1→1, 3→3}
- keyToFreq: {1→2, 3→1}
- freqToKeys: {1→{3}, 2→{1}}
- minFreq: 1

get(2):
- Key 2 was evicted
- Returns: -1
```

## 📊 Why O(1) for All Operations?

### The Magic of Three Hash Maps

1. **`get` operation**:
   - Map lookup: O(1)
   - Frequency update: O(1) set operations
   
2. **`put` operation**:
   - Map insertions: O(1)
   - Eviction (if needed): O(1) using minFreq
   
3. **`evict` operation**:
   - Find minimum frequency: O(1) - we track minFreq
   - Get LRU from that frequency: O(1) - first in Set
   - Delete from maps: O(1)

### Critical Design Choices

```javascript
// Why Set for frequency buckets?
freqToKeys: Map<number, Set<string>>
// 1. Maintains insertion order (LRU tracking)
// 2. O(1) add/delete operations
// 3. O(1) access to first element

// Why track minFreq separately?
// Without it, finding minimum would be O(n)
// With it, eviction is O(1)
```

## 🚫 Common Mistakes

1. **Not Maintaining minFreq Correctly**
   ```javascript
   // Wrong: Forgetting to update when bucket empties
   if (oldFreqKeys.size === 0) {
       this.freqToKeys.delete(freq);
       // Missing: if (this.minFreq === freq) this.minFreq++;
   }
   ```

2. **Using Array Instead of Set for Frequency Buckets**
   ```javascript
   // Wrong: Array operations are O(n)
   freqToKeys.set(freq, []); // Array
   array.splice(array.indexOf(key), 1); // O(n) deletion!
   
   // Right: Set operations are O(1)
   freqToKeys.set(freq, new Set());
   set.delete(key); // O(1)
   ```

3. **Not Handling Zero Capacity**
   ```javascript
   // Always check at the start of put()
   if (this.capacity === 0) return;
   ```

## 🔄 Alternative: Using Doubly Linked Lists

```javascript
class Node {
    constructor(key, val, freq = 1) {
        this.key = key;
        this.val = val;
        this.freq = freq;
        this.prev = null;
        this.next = null;
    }
}

class DoublyLinkedList {
    constructor() {
        this.head = new Node(0, 0);
        this.tail = new Node(0, 0);
        this.head.next = this.tail;
        this.tail.prev = this.head;
        this.size = 0;
    }
    
    addToHead(node) {
        node.next = this.head.next;
        node.prev = this.head;
        this.head.next.prev = node;
        this.head.next = node;
        this.size++;
    }
    
    removeNode(node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
        this.size--;
    }
    
    removeTail() {
        if (this.size > 0) {
            const node = this.tail.prev;
            this.removeNode(node);
            return node;
        }
        return null;
    }
}

// LFU implementation using DLL for each frequency
class LFUCacheWithDLL {
    constructor(capacity) {
        this.capacity = capacity;
        this.minFreq = 0;
        this.keyToNode = new Map();
        this.freqToList = new Map();
    }
    
    // Similar logic but using DLL for O(1) operations
}
```

## 🔗 Related Problems

- **LRU Cache** - Simpler version using only recency
- **All O(1) Data Structure** - Similar multi-map design
- **Design Twitter** - Uses similar frequency tracking
- **Top K Frequent Elements** - Frequency counting pattern

## 💡 Key Takeaways

1. **Multiple Hash Maps**: Complex O(1) operations often need multiple data structures
2. **Track Metadata**: minFreq saves us from O(n) searches
3. **Sets Maintain Order**: JavaScript Sets remember insertion order
4. **Design for Edge Cases**: Zero capacity, single element, etc.

This is a classic system design problem that tests your ability to optimize for specific constraints!
