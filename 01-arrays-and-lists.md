# Arrays and Lists

## Plain English Explanation
Think of an **array** like a row of numbered lockers in a hallway. Each locker has a number (index), and you can go directly to any locker if you know its number. This is why accessing array elements is fast - you don't have to check other lockers first.

A **linked list** is like a treasure hunt where each clue tells you where the next clue is. You have to follow the clues in order - you can't skip ahead. But adding a new clue in the middle is easy - you just change what the previous clue points to.

## Visual Representation
```
Array (Contiguous Memory):
Index:    [0]  [1]  [2]  [3]  [4]
Values:   [10] [20] [30] [40] [50]
Memory:   100  104  108  112  116  (addresses)
          ↑ Direct access by calculating: base + (index × size)

Linked List (Scattered Memory):
[10|→] → [20|→] → [30|→] → [40|→] → [50|X]
 Head                                 Tail
 ↑ Must traverse from head to reach any element
```

## Overview
Arrays and lists are fundamental data structures that form the backbone of most algorithms. Understanding their performance characteristics is crucial for system design and optimization.

### Key Concepts
- **Dynamic Arrays**: Resizable arrays with amortized O(1) append
- **Linked Lists**: Sequential access structures with O(1) insertion/deletion
- **AWS Applications**: DynamoDB partition keys, S3 object listings, Lambda event batches

### When to Use What?
- **Use Arrays when**: You need random access, know the size beforehand, or need cache-friendly operations
- **Use Linked Lists when**: Frequent insertion/deletion at arbitrary positions, unknown size, memory is fragmented
- **Avoid Arrays when**: Frequent insertion at beginning, size varies dramatically
- **Avoid Linked Lists when**: Need random access, cache performance matters

## JavaScript Arrays - Dynamic Implementation

### Core Operations & Complexity

```javascript
// JavaScript arrays are dynamic by default
const arr = []; // Creates empty dynamic array with initial capacity

// ============= ACCESS OPERATION =============
// Time: O(1) - WHY?
// Arrays store elements in contiguous memory
// Address calculation: base_address + (index × element_size)
// This is a simple arithmetic operation, always takes same time
console.log(arr[0]); // Direct memory access - instant lookup

// ============= PUSH (APPEND) OPERATION =============
// Time: O(1) amortized - WHY?
// Most pushes just add to end: O(1)
// Occasionally needs resize (double capacity): O(n) to copy all elements
// But resize happens at sizes 2, 4, 8, 16, 32...
// Total cost for n pushes: n + n/2 + n/4 + ... ≈ 2n
// Average per operation: 2n/n = 2 = O(1)
arr.push(5); // Adds element to arr[length], increments length

// ============= POP OPERATION =============
// Time: O(1) - WHY?
// Simply decrements length and returns last element
// No shifting needed, no other elements affected
arr.pop(); // arr.length-- and return arr[arr.length]

// ============= UNSHIFT (PREPEND) OPERATION =============
// Time: O(n) - WHY?
// Must shift EVERY existing element one position right
// If we have 1000 elements, we move all 1000
// More elements = proportionally more work
arr.unshift(1); // Shifts all n elements, then inserts at [0]

// ============= SHIFT OPERATION =============
// Time: O(n) - WHY?
// Removes first element, must shift all others left
// arr[0] = arr[1], arr[1] = arr[2], ... for all n-1 elements
arr.shift(); // Removes arr[0], shifts all n-1 elements left

// ============= INSERT AT INDEX OPERATION =============
// Time: O(n) - WHY?
// Must shift all elements after insertion point
// Worst case: insert at beginning = shift all n elements
// Best case: insert at end = shift 0 elements (but that's push!)
// Average: shift n/2 elements = O(n)
arr.splice(2, 0, 'inserted'); // Shifts elements from index 2 onwards

// ============= SEARCH OPERATIONS =============
// Time: O(n) - WHY?
// No inherent order (unless sorted), must check each element
// Worst case: element is last or doesn't exist = check all n
// Average case: check n/2 elements = O(n)
arr.indexOf(item);  // Checks arr[0], arr[1], arr[2]... until found
arr.includes(item); // Same as indexOf, returns boolean

// ============= SPACE COMPLEXITY =============
// Space: O(n) - WHY?
// Stores n elements
// May have some unused capacity (up to 2n after resize)
// But we ignore constants, so O(2n) = O(n)
```

### Dynamic Array Implementation from Scratch

```javascript
class DynamicArray {
    constructor() {
        this.data = {};
        this.length = 0;
        this.capacity = 2; // Initial capacity
    }
    
    // O(1) amortized
    push(item) {
        if (this.length === this.capacity) {
            this.resize(); // Double capacity when full
        }
        this.data[this.length] = item;
        this.length++;
    }
    
    // O(n) - but happens rarely
    resize() {
        const newData = {};
        const newCapacity = this.capacity * 2;
        
        for (let i = 0; i < this.length; i++) {
            newData[i] = this.data[i];
        }
        
        this.data = newData;
        this.capacity = newCapacity;
    }
    
    // O(1)
    get(index) {
        if (index < 0 || index >= this.length) {
            throw new Error('Index out of bounds');
        }
        return this.data[index];
    }
    
    // O(1)
    pop() {
        if (this.length === 0) return undefined;
        const item = this.data[this.length - 1];
        delete this.data[this.length - 1];
        this.length--;
        return item;
    }
    
    // O(n) - shifts all elements
    unshift(item) {
        for (let i = this.length; i > 0; i--) {
            this.data[i] = this.data[i - 1];
        }
        this.data[0] = item;
        this.length++;
    }
}

// Usage example
const myArray = new DynamicArray();
myArray.push(1); // O(1)
myArray.push(2); // O(1)
myArray.push(3); // Triggers resize, still O(1) amortized
console.log(myArray.get(1)); // 2, O(1)
```

## Linked Lists

### Singly Linked List Implementation

```javascript
class Node {
    constructor(value) {
        this.value = value;
        this.next = null;
    }
}

class LinkedList {
    constructor() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }
    
    // O(1) - insert at head
    prepend(value) {
        const newNode = new Node(value);
        newNode.next = this.head;
        this.head = newNode;
        
        if (!this.tail) {
            this.tail = newNode;
        }
        this.size++;
    }
    
    // O(1) - insert at tail
    append(value) {
        const newNode = new Node(value);
        
        if (!this.head) {
            this.head = newNode;
            this.tail = newNode;
        } else {
            this.tail.next = newNode;
            this.tail = newNode;
        }
        this.size++;
    }
    
    // O(n) - search for value
    find(value) {
        let current = this.head;
        while (current) {
            if (current.value === value) {
                return current;
            }
            current = current.next;
        }
        return null;
    }
    
    // O(n) - delete by value
    delete(value) {
        if (!this.head) return false;
        
        if (this.head.value === value) {
            this.head = this.head.next;
            this.size--;
            if (this.size === 0) {
                this.tail = null;
            }
            return true;
        }
        
        let current = this.head;
        while (current.next) {
            if (current.next.value === value) {
                current.next = current.next.next;
                if (current.next === null) {
                    this.tail = current;
                }
                this.size--;
                return true;
            }
            current = current.next;
        }
        return false;
    }
    
    // O(n) - convert to array
    toArray() {
        const result = [];
        let current = this.head;
        while (current) {
            result.push(current.value);
            current = current.next;
        }
        return result;
    }
}

// Usage
const list = new LinkedList();
list.append(1);    // O(1)
list.append(2);    // O(1)
list.prepend(0);   // O(1)
console.log(list.toArray()); // [0, 1, 2]
```

### Doubly Linked List

```javascript
class DoublyNode {
    constructor(value) {
        this.value = value;
        this.next = null;
        this.prev = null;
    }
}

class DoublyLinkedList {
    constructor() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }
    
    // O(1) - insert at position
    insertAt(index, value) {
        if (index < 0 || index > this.size) {
            throw new Error('Index out of bounds');
        }
        
        const newNode = new DoublyNode(value);
        
        if (index === 0) {
            newNode.next = this.head;
            if (this.head) {
                this.head.prev = newNode;
            }
            this.head = newNode;
            if (!this.tail) {
                this.tail = newNode;
            }
        } else if (index === this.size) {
            this.tail.next = newNode;
            newNode.prev = this.tail;
            this.tail = newNode;
        } else {
            // O(n) to find position
            let current = this.head;
            for (let i = 0; i < index - 1; i++) {
                current = current.next;
            }
            newNode.next = current.next;
            newNode.prev = current;
            current.next.prev = newNode;
            current.next = newNode;
        }
        
        this.size++;
    }
    
    // O(1) if node reference is given
    removeNode(node) {
        if (node.prev) {
            node.prev.next = node.next;
        } else {
            this.head = node.next;
        }
        
        if (node.next) {
            node.next.prev = node.prev;
        } else {
            this.tail = node.prev;
        }
        
        this.size--;
    }
}
```

## Array vs Linked List Comparison

| Operation | Array | Linked List | Use Array When | Use Linked List When |
|-----------|-------|-------------|----------------|---------------------|
| Access by Index | O(1) | O(n) | Random access needed | Sequential access only |
| Insert at Beginning | O(n) | O(1) | Rarely prepending | Frequent prepending |
| Insert at End | O(1)* | O(1) | Default choice | With tail pointer |
| Insert at Middle | O(n) | O(1)** | Infrequent inserts | Frequent inserts |
| Delete | O(n) | O(1)** | Infrequent deletes | Frequent deletes |
| Memory | Contiguous | Scattered | Cache efficiency matters | Dynamic size varies greatly |

*Amortized, **With node reference

## AWS Applications

### 1. DynamoDB Partition Keys
```javascript
// DynamoDB uses arrays internally for partition management
const partitionKey = {
    // Array-like structure for item distribution
    hashFunction: (key) => {
        // Returns partition array index
        return hash(key) % numberOfPartitions;
    },
    
    // Each partition is essentially an array of items
    partitions: [
        [/* items */],
        [/* items */],
        // ...
    ]
};

// Optimal partition key selection
const goodPartitionKey = {
    userId: "user123", // High cardinality, even distribution
};

const badPartitionKey = {
    status: "active", // Low cardinality, hot partitions
};
```

### 2. S3 Object Listing
```javascript
// S3 ListObjects returns array-like paginated results
async function listAllS3Objects(bucket, prefix) {
    const objects = []; // Dynamic array for results
    let continuationToken = null;
    
    do {
        const response = await s3.listObjectsV2({
            Bucket: bucket,
            Prefix: prefix,
            MaxKeys: 1000, // Array chunk size
            ContinuationToken: continuationToken
        }).promise();
        
        objects.push(...response.Contents); // O(n) operation
        continuationToken = response.NextContinuationToken;
    } while (continuationToken);
    
    return objects;
}
```

### 3. Lambda Batch Processing
```javascript
// Lambda processes arrays of events efficiently
exports.handler = async (event) => {
    // Event.Records is an array
    const batchSize = event.Records.length;
    
    // Process in chunks for memory efficiency
    const chunkSize = 25;
    const results = [];
    
    for (let i = 0; i < batchSize; i += chunkSize) {
        const chunk = event.Records.slice(i, i + chunkSize); // O(k) where k is chunk size
        const chunkResults = await Promise.all(
            chunk.map(record => processRecord(record))
        );
        results.push(...chunkResults);
    }
    
    return {
        batchItemFailures: results
            .filter(r => r.failed)
            .map(r => ({ itemIdentifier: r.id }))
    };
};
```

## Practice Problems

### Problem 1: Two Sum
```javascript
/**
 * Find two numbers in array that sum to target
 * Time: O(n), Space: O(n)
 */
function twoSum(nums, target) {
    const seen = new Map(); // Hash table for O(1) lookup
    
    for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];
        if (seen.has(complement)) {
            return [seen.get(complement), i];
        }
        seen.set(nums[i], i);
    }
    
    return [];
}

// Test
console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
```

### Problem 2: Merge Two Sorted Lists
```javascript
/**
 * Merge two sorted linked lists
 * Time: O(n + m), Space: O(1)
 */
function mergeTwoLists(l1, l2) {
    const dummy = new Node(0);
    let current = dummy;
    
    while (l1 && l2) {
        if (l1.value <= l2.value) {
            current.next = l1;
            l1 = l1.next;
        } else {
            current.next = l2;
            l2 = l2.next;
        }
        current = current.next;
    }
    
    // Attach remaining nodes
    current.next = l1 || l2;
    
    return dummy.next;
}
```

### Problem 3: LRU Cache (Using Doubly Linked List + Hash Map)
```javascript
/**
 * Least Recently Used Cache
 * All operations O(1)
 */
class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.map = new Map(); // Hash map for O(1) access
        this.head = new DoublyNode(0);
        this.tail = new DoublyNode(0);
        this.head.next = this.tail;
        this.tail.prev = this.head;
    }
    
    get(key) {
        if (!this.map.has(key)) return -1;
        
        const node = this.map.get(key);
        this.moveToHead(node);
        return node.value;
    }
    
    put(key, value) {
        if (this.map.has(key)) {
            const node = this.map.get(key);
            node.value = value;
            this.moveToHead(node);
        } else {
            const node = new DoublyNode(value);
            node.key = key;
            this.map.set(key, node);
            this.addToHead(node);
            
            if (this.map.size > this.capacity) {
                const removed = this.removeTail();
                this.map.delete(removed.key);
            }
        }
    }
    
    moveToHead(node) {
        this.removeNode(node);
        this.addToHead(node);
    }
    
    addToHead(node) {
        node.prev = this.head;
        node.next = this.head.next;
        this.head.next.prev = node;
        this.head.next = node;
    }
    
    removeNode(node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    removeTail() {
        const node = this.tail.prev;
        this.removeNode(node);
        return node;
    }
}
```

## Interview Tips

### Common Questions
1. **"Why would you use a linked list over an array?"**
   - Frequent insertions/deletions at arbitrary positions
   - Unknown or highly variable size
   - Memory is fragmented

2. **"How does JavaScript handle array resizing?"**
   - V8 engine uses growth factor of ~1.5
   - Allocates extra capacity to minimize resizing
   - Old array is garbage collected after copying

3. **"Design a data structure with O(1) insert, delete, and getRandom"**
   - Use array + hash map combination
   - Array stores values, map stores value->index

### Red Flags to Avoid
- ❌ Using array.shift() in a loop (O(n²) complexity)
- ❌ Not considering memory locality for cache performance
- ❌ Forgetting to update tail pointer in linked list operations
- ❌ Not handling edge cases (empty list, single element)

### AWS-Specific Follow-ups
- "How would you implement pagination for large datasets?"
- "Design a distributed array across multiple EC2 instances"
- "How does DynamoDB handle array attributes?"
- "Optimize Lambda function processing large arrays"

## Quiz Questions (With Answers)

### Q1: Why is accessing an array element O(1) while accessing a linked list element is O(n)?

**Answer:** Arrays store elements in contiguous memory locations. To access element at index `i`, we calculate its memory address using the formula: `base_address + (i × element_size)`. This is a simple arithmetic operation that takes constant time.

Linked lists store elements in scattered memory locations connected by pointers. To reach the nth element, we must traverse through all n-1 previous elements by following the chain of pointers. There's no way to "jump" directly to a middle element.

### Q2: What does "amortized O(1)" mean for array push operations?

**Answer:** Amortized O(1) means that while individual operations might occasionally take O(n) time (when resizing), the average time per operation over a sequence of operations is O(1). 

When an array is full and needs resizing:
- One push operation takes O(n) to copy all elements to new array
- But this happens rarely (every time capacity doubles)
- If we push n elements total, resize happens log(n) times
- Total cost: n + n/2 + n/4 + ... ≈ 2n
- Average per operation: 2n/n = O(1)

### Q3: In what scenario would you choose a linked list over an array in a production system?

**Answer:** Choose a linked list when:
1. **Frequent insertions/deletions in the middle**: Like maintaining a sorted list with frequent updates
2. **Implementation of undo/redo**: Each state links to previous/next
3. **Memory is highly fragmented**: Can't allocate large contiguous blocks
4. **Building a queue with no size limit**: Using head/tail pointers
5. **LRU cache implementation**: Combined with hash map for O(1) operations

Real AWS example: DynamoDB uses linked lists in its internal storage engine for managing item versions and maintaining sort order within partitions.

### Q4: What's the space complexity difference between arrays and linked lists?

**Answer:** 
- **Array**: O(n) for n elements - just the data
- **Linked List**: O(n) but with higher constant factor - data + pointer(s)

In practice:
- Array of integers (4 bytes each): 4n bytes total
- Linked list of integers: 4n bytes (data) + 8n bytes (pointers on 64-bit system) = 12n bytes
- **Linked lists use approximately 3x more memory** for simple data types

### Q5: How would you detect a cycle in a linked list?

**Answer:** Use Floyd's Cycle Detection Algorithm (Tortoise and Hare):

```javascript
function hasCycle(head) {
    let slow = head;
    let fast = head;
    
    while (fast && fast.next) {
        slow = slow.next;        // Move 1 step
        fast = fast.next.next;   // Move 2 steps
        
        if (slow === fast) {
            return true;  // Cycle detected
        }
    }
    
    return false;  // No cycle
}
```

**Why it works:** If there's a cycle, the fast pointer will eventually "lap" the slow pointer inside the cycle. Time: O(n), Space: O(1).

## Common Mistakes to Avoid

1. **Array Resizing in a Loop**
   ```javascript
   // BAD: O(n²) due to repeated resizing
   for (let i = 0; i < n; i++) {
       arr.unshift(i);  // Shifts all elements each time
   }
   
   // GOOD: O(n) - build reversed then reverse
   for (let i = 0; i < n; i++) {
       arr.push(i);
   }
   arr.reverse();
   ```

2. **Not Updating Pointers Correctly in Linked Lists**
   ```javascript
   // BAD: Lost reference to rest of list
   node.next = newNode;
   
   // GOOD: Save reference first
   newNode.next = node.next;
   node.next = newNode;
   ```

3. **Forgetting Edge Cases**
   - Empty array/list
   - Single element
   - Inserting at boundaries (index 0 or length)
   - Null pointer checks in linked lists

## Key Takeaways
1. **Arrays** excel at random access and cache locality
2. **Linked Lists** excel at insertion/deletion with known positions
3. **Hybrid approaches** (like deque) often provide best of both worlds
4. **AWS services** heavily utilize array-based structures for performance
5. **Memory patterns** matter more in distributed systems
6. **Always consider cache performance** in modern systems

---

*Next: [Trees and Graphs →](./02-trees-and-graphs.md)*
