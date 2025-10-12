# LRU Cache (Least Recently Used Cache)

## 🎯 What Are We Trying To Solve?

Imagine your phone can only keep 3 apps in memory. When you open a 4th app, it needs to remove one. Which one should it remove? The app you haven't used in the longest time!

This is an LRU (Least Recently Used) Cache - it keeps track of what you've used recently and removes the least recently used item when it runs out of space.

Real-world examples:
- Your browser's back button (keeps recent pages)
- Operating system's page cache
- Database buffer pools
- CDN edge servers

## 📝 Examples

**Example with capacity = 2:**
```
LRUCache cache = new LRUCache(2);  // Can hold 2 items

cache.put(1, 1);    // Cache: {1=1}
cache.put(2, 2);    // Cache: {1=1, 2=2}
cache.get(1);       // Returns 1, marks key 1 as recently used
cache.put(3, 3);    // Evicts key 2 (least recently used), Cache: {1=1, 3=3}
cache.get(2);       // Returns -1 (not found)
cache.put(4, 4);    // Evicts key 1, Cache: {3=3, 4=4}
cache.get(1);       // Returns -1 (not found)
cache.get(3);       // Returns 3
cache.get(4);       // Returns 4
```

## 🧠 How Do We Think About This?

We need two things:
1. **Fast lookups** (O(1)) - Use a hash map
2. **Track usage order** (O(1)) - Use a doubly linked list

The trick: Combine both!
- Hash map points to nodes in the linked list
- Linked list maintains order (most recent at head, least recent at tail)

Think of it like a line at a store:
- When someone is served (accessed), they go to the front of the line
- When the store is full, the person at the back leaves

## 💻 The Code

```javascript
// Node for our doubly linked list
class Node {
    constructor(key = 0, value = 0) {
        this.key = key;      // We need key for deletion from map
        this.value = value;
        this.prev = null;    // Pointer to previous node
        this.next = null;    // Pointer to next node
    }
}

class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;  // Maximum items we can store
        this.map = new Map();       // For O(1) lookups: key -> node
        
        // Dummy head and tail make operations easier
        // Real nodes go between them
        this.head = new Node();  // Most recently used side
        this.tail = new Node();  // Least recently used side
        
        // Connect dummy nodes
        this.head.next = this.tail;
        this.tail.prev = this.head;
    }
    
    // Get value and mark as recently used
    get(key) {
        // Check if key exists
        if (!this.map.has(key)) {
            return -1;  // Not found
        }
        
        // Get the node from our map
        const node = this.map.get(key);
        
        // Move to head (mark as recently used)
        this.moveToHead(node);
        
        return node.value;
    }
    
    // Add or update a key-value pair
    put(key, value) {
        // If key exists, update it
        if (this.map.has(key)) {
            const node = this.map.get(key);
            node.value = value;  // Update value
            this.moveToHead(node);  // Mark as recently used
            return;
        }
        
        // Create new node
        const newNode = new Node(key, value);
        
        // Add to map and list
        this.map.set(key, newNode);
        this.addToHead(newNode);
        
        // Check if we exceeded capacity
        if (this.map.size > this.capacity) {
            // Remove least recently used (at tail)
            const removed = this.removeTail();
            this.map.delete(removed.key);  // Don't forget to remove from map!
        }
    }
    
    // Helper: Move node to head (mark as most recently used)
    moveToHead(node) {
        this.removeNode(node);  // Remove from current position
        this.addToHead(node);   // Add to head
    }
    
    // Helper: Add node right after head
    addToHead(node) {
        // Insert between head and head.next
        node.prev = this.head;
        node.next = this.head.next;
        
        // Update pointers
        this.head.next.prev = node;
        this.head.next = node;
    }
    
    // Helper: Remove node from list
    removeNode(node) {
        // Connect neighbors to each other
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    // Helper: Remove and return tail node (least recently used)
    removeTail() {
        const node = this.tail.prev;  // Real tail (not dummy)
        this.removeNode(node);
        return node;
    }
}

// Test the cache
const cache = new LRUCache(2);

cache.put(1, 1);
console.log("Put (1,1), Cache state: 1");

cache.put(2, 2);
console.log("Put (2,2), Cache state: 2 -> 1");

console.log(cache.get(1));  // Returns 1
console.log("Get (1), Cache state: 1 -> 2");

cache.put(3, 3);  // Evicts key 2
console.log("Put (3,3), Cache state: 3 -> 1");

console.log(cache.get(2));  // Returns -1 (not found)

cache.put(4, 4);  // Evicts key 1
console.log("Put (4,4), Cache state: 4 -> 3");

console.log(cache.get(1));  // Returns -1 (not found)
console.log(cache.get(3));  // Returns 3
console.log(cache.get(4));  // Returns 4
```

## 🎨 Visual Walkthrough

Let's visualize the cache with capacity = 2:

```
Initial state:
  Map: {}
  List: [head] ⟷ [tail]

After put(1, 1):
  Map: {1 → node1}
  List: [head] ⟷ [1] ⟷ [tail]
                  ↑ Most recent

After put(2, 2):
  Map: {1 → node1, 2 → node2}
  List: [head] ⟷ [2] ⟷ [1] ⟷ [tail]
                  ↑ Most recent

After get(1):
  Map: {1 → node1, 2 → node2}
  List: [head] ⟷ [1] ⟷ [2] ⟷ [tail]
                  ↑ Most recent (moved!)

After put(3, 3):
  Map: {1 → node1, 3 → node3}  (2 removed!)
  List: [head] ⟷ [3] ⟷ [1] ⟷ [tail]
                  ↑ Most recent
```

## ⏱️ How Fast Is This?

All operations are **O(1)**:
- **get(key)**: O(1)
  - Hash map lookup: O(1)
  - Move to head: O(1) (just pointer updates)
  
- **put(key, value)**: O(1)
  - Hash map insert: O(1)
  - Add to head: O(1)
  - Remove tail if needed: O(1)

- **Space Complexity**: O(capacity)
  - We store at most 'capacity' items

## 🚫 Common Mistakes

1. **Forgetting to store the key in the node**
   ```javascript
   // Wrong: Can't remove from map when evicting
   class Node {
       constructor(value) {
           this.value = value;  // Missing key!
       }
   }
   
   // Right: Store both key and value
   class Node {
       constructor(key, value) {
           this.key = key;    // Need this for map deletion
           this.value = value;
       }
   }
   ```

2. **Not updating the map when evicting**
   ```javascript
   // Wrong: Only removes from list
   const removed = this.removeTail();
   
   // Right: Remove from both list AND map
   const removed = this.removeTail();
   this.map.delete(removed.key);
   ```

3. **Incorrect order maintenance**
   ```javascript
   // Wrong: Forgets to move to head on get()
   get(key) {
       return this.map.get(key).value;
   }
   
   // Right: Move to head to mark as recently used
   get(key) {
       const node = this.map.get(key);
       this.moveToHead(node);  // Don't forget this!
       return node.value;
   }
   ```

## 🔄 Alternative Approaches

### Using JavaScript's Map (Maintains Insertion Order)
```javascript
class SimpleLRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.cache = new Map();  // Maps maintain insertion order
    }
    
    get(key) {
        if (!this.cache.has(key)) return -1;
        
        // Move to end (most recent)
        const value = this.cache.get(key);
        this.cache.delete(key);
        this.cache.set(key, value);
        return value;
    }
    
    put(key, value) {
        // Remove if exists (to update position)
        if (this.cache.has(key)) {
            this.cache.delete(key);
        }
        
        // Add to end (most recent)
        this.cache.set(key, value);
        
        // Remove oldest if over capacity
        if (this.cache.size > this.capacity) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
    }
}
```
Note: This is simpler but the delete + re-insert isn't truly O(1) in all JavaScript engines.

## 🔗 Related Problems

- **LFU Cache**: Least Frequently Used (tracks frequency, not recency)
- **Design HashMap**: Implement a hash map from scratch
- **Design LinkedList**: Implement a linked list
- **Max Stack**: Stack that can return maximum element

## 💡 Key Takeaway

The LRU Cache is a perfect example of combining data structures for optimal performance. By using both a hash map (for fast lookups) and a doubly linked list (for order tracking), we achieve O(1) for all operations. This pattern appears in real systems everywhere - from CPU caches to web servers!
