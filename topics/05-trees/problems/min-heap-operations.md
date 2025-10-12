# Min Heap Operations

## 🎯 What Are We Trying To Solve?

A heap is like a priority line where the most important person (minimum or maximum value) is always at the front. We need to implement basic heap operations: insert, delete, and find minimum - the foundation of priority queues!

## 📝 The Problem

Implement a min-heap data structure that supports three operations:
1. **Insert**: Add an element to the heap
2. **Delete**: Remove a specific element from the heap
3. **Get Minimum**: Return the smallest element (root of min-heap)

The heap property must be maintained: parent ≤ all children.

```
Example Operations:
Insert 4: [4]
Insert 9: [4, 9]
Insert 2: [2, 9, 4]  (2 bubbles up)
Get Min: 2
Delete 9: [2, 4]
Get Min: 2
```

## 💻 The Code

```javascript
/**
 * MinHeap implementation with array
 */
class MinHeap {
    constructor() {
        this.heap = [];
    }
    
    /**
     * Get parent, left child, right child indices
     */
    getParentIndex(i) { return Math.floor((i - 1) / 2); }
    getLeftChildIndex(i) { return 2 * i + 1; }
    getRightChildIndex(i) { return 2 * i + 2; }
    
    hasParent(i) { return this.getParentIndex(i) >= 0; }
    hasLeftChild(i) { return this.getLeftChildIndex(i) < this.heap.length; }
    hasRightChild(i) { return this.getRightChildIndex(i) < this.heap.length; }
    
    parent(i) { return this.heap[this.getParentIndex(i)]; }
    leftChild(i) { return this.heap[this.getLeftChildIndex(i)]; }
    rightChild(i) { return this.heap[this.getRightChildIndex(i)]; }
    
    /**
     * Swap two elements in the heap
     */
    swap(i, j) {
        [this.heap[i], this.heap[j]] = [this.heap[j], this.heap[i]];
    }
    
    /**
     * Insert element - O(log n)
     */
    insert(value) {
        this.heap.push(value);
        this.heapifyUp();
    }
    
    /**
     * Bubble up the last element
     */
    heapifyUp() {
        let index = this.heap.length - 1;
        
        while (this.hasParent(index) && this.parent(index) > this.heap[index]) {
            const parentIndex = this.getParentIndex(index);
            this.swap(index, parentIndex);
            index = parentIndex;
        }
    }
    
    /**
     * Get minimum element - O(1)
     */
    getMin() {
        if (this.heap.length === 0) {
            throw new Error("Heap is empty");
        }
        return this.heap[0];
    }
    
    /**
     * Extract minimum element - O(log n)
     */
    extractMin() {
        if (this.heap.length === 0) {
            throw new Error("Heap is empty");
        }
        
        const min = this.heap[0];
        this.heap[0] = this.heap[this.heap.length - 1];
        this.heap.pop();
        
        if (this.heap.length > 0) {
            this.heapifyDown();
        }
        
        return min;
    }
    
    /**
     * Bubble down from root
     */
    heapifyDown() {
        let index = 0;
        
        while (this.hasLeftChild(index)) {
            let smallerChildIndex = this.getLeftChildIndex(index);
            
            if (this.hasRightChild(index) && 
                this.rightChild(index) < this.leftChild(index)) {
                smallerChildIndex = this.getRightChildIndex(index);
            }
            
            if (this.heap[index] <= this.heap[smallerChildIndex]) {
                break;
            }
            
            this.swap(index, smallerChildIndex);
            index = smallerChildIndex;
        }
    }
    
    /**
     * Delete specific element - O(n + log n)
     */
    delete(value) {
        const index = this.heap.indexOf(value);
        if (index === -1) return false;
        
        // Replace with last element
        this.heap[index] = this.heap[this.heap.length - 1];
        this.heap.pop();
        
        if (index < this.heap.length) {
            // Try both heapify operations
            const parentIdx = this.getParentIndex(index);
            if (parentIdx >= 0 && this.heap[parentIdx] > this.heap[index]) {
                this.heapifyUpFrom(index);
            } else {
                this.heapifyDownFrom(index);
            }
        }
        
        return true;
    }
    
    heapifyUpFrom(index) {
        while (this.hasParent(index) && this.parent(index) > this.heap[index]) {
            const parentIndex = this.getParentIndex(index);
            this.swap(index, parentIndex);
            index = parentIndex;
        }
    }
    
    heapifyDownFrom(index) {
        while (this.hasLeftChild(index)) {
            let smallerChildIndex = this.getLeftChildIndex(index);
            
            if (this.hasRightChild(index) && 
                this.rightChild(index) < this.leftChild(index)) {
                smallerChildIndex = this.getRightChildIndex(index);
            }
            
            if (this.heap[index] <= this.heap[smallerChildIndex]) {
                break;
            }
            
            this.swap(index, smallerChildIndex);
            index = smallerChildIndex;
        }
    }
    
    /**
     * Build heap from array - O(n)
     */
    static buildHeap(array) {
        const heap = new MinHeap();
        heap.heap = [...array];
        
        // Start from last non-leaf node
        for (let i = Math.floor(array.length / 2) - 1; i >= 0; i--) {
            heap.heapifyDownFrom(i);
        }
        
        return heap;
    }
    
    size() {
        return this.heap.length;
    }
    
    isEmpty() {
        return this.heap.length === 0;
    }
}

/**
 * Alternative: Using JavaScript's built-in sort for simplicity
 * (Not a true heap but works for the operations)
 */
class SimpleMinHeap {
    constructor() {
        this.data = [];
    }
    
    insert(value) {
        this.data.push(value);
        this.data.sort((a, b) => a - b);
    }
    
    delete(value) {
        const index = this.data.indexOf(value);
        if (index !== -1) {
            this.data.splice(index, 1);
        }
    }
    
    getMin() {
        return this.data[0];
    }
}

/**
 * Solution for the original problem
 */
function processHeapOperations(operations) {
    const heap = new MinHeap();
    const results = [];
    
    for (const op of operations) {
        const [type, value] = op;
        
        switch(type) {
            case 1: // Insert
                heap.insert(value);
                break;
            case 2: // Delete
                heap.delete(value);
                break;
            case 3: // Get minimum
                results.push(heap.getMin());
                break;
        }
    }
    
    return results;
}

// Test cases
const heap = new MinHeap();
heap.insert(4);
heap.insert(9);
heap.insert(2);
console.log(heap.getMin()); // 2
heap.delete(9);
console.log(heap.getMin()); // 2
heap.insert(1);
console.log(heap.getMin()); // 1

// Process operations
const operations = [
    [1, 4],  // Insert 4
    [1, 9],  // Insert 9
    [3],     // Get min → 4
    [1, 2],  // Insert 2
    [3],     // Get min → 2
    [2, 4],  // Delete 4
    [3]      // Get min → 2
];
console.log(processHeapOperations(operations)); // [4, 2, 2]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through inserting and deleting in a min-heap:

```
Insert sequence: 4, 9, 2, 7, 5

Step 1: Insert 4
    4
    
Step 2: Insert 9
    4
   /
  9

Step 3: Insert 2
    4           2 bubbles up
   /    →      /
  9 2         9 4
  
Step 4: Insert 7
    2
   / \
  9   4
 /
7

Step 5: Insert 5
    2           2
   / \         / \
  9   4   →   5   4
 / \         / \
7   5       7   9
(5 bubbles up, swaps with 9)

Delete 5:
    2           2
   / \         / \
  5   4   →   9   4
 / \         /
7   9       7
(Replace 5 with last element 9, heapify down)
```

## 📊 Visual Understanding

```
Heap as Array vs Tree:

Array: [2, 5, 4, 7, 9]
Index:  0  1  2  3  4

Tree:       2 (0)
          /   \
       5(1)    4(2)
       /  \
     7(3)  9(4)

Parent of i: floor((i-1)/2)
Left child:  2*i + 1
Right child: 2*i + 2
```

## ⏱️ Complexity Analysis

| Operation | Time Complexity | Space Complexity |
|-----------|----------------|------------------|
| Insert | O(log n) | O(1) |
| Delete (specific) | O(n) | O(1) |
| Extract Min | O(log n) | O(1) |
| Get Min | O(1) | O(1) |
| Build Heap | O(n) | O(1) |

## 🚫 Common Mistakes

1. **Wrong parent/child index calculations**
   ```javascript
   // Wrong: Off by one
   getParentIndex(i) { return Math.floor(i / 2); }
   
   // Right: Account for 0-based indexing
   getParentIndex(i) { return Math.floor((i - 1) / 2); }
   ```

2. **Not maintaining heap property after delete**
   ```javascript
   // Wrong: Just remove element
   delete(value) {
       const index = this.heap.indexOf(value);
       this.heap.splice(index, 1);
   }
   
   // Right: Replace and heapify
   delete(value) {
       // Replace with last, then heapify
   }
   ```

3. **Incorrect heapify direction**
   ```javascript
   // After delete, need to check both directions
   if (newValue < parent) {
       heapifyUp();
   } else {
       heapifyDown();
   }
   ```

## 🔄 Variations

```javascript
// Max Heap (just reverse comparisons)
class MaxHeap extends MinHeap {
    heapifyUp() {
        let index = this.heap.length - 1;
        while (this.hasParent(index) && this.parent(index) < this.heap[index]) {
            const parentIndex = this.getParentIndex(index);
            this.swap(index, parentIndex);
            index = parentIndex;
        }
    }
}

// Priority Queue with custom comparator
class PriorityQueue {
    constructor(compareFn = (a, b) => a - b) {
        this.heap = [];
        this.compare = compareFn;
    }
    
    insert(value) {
        this.heap.push(value);
        this.heapifyUp();
    }
    
    heapifyUp() {
        let index = this.heap.length - 1;
        while (this.hasParent(index)) {
            const parentIndex = this.getParentIndex(index);
            if (this.compare(this.heap[parentIndex], this.heap[index]) <= 0) {
                break;
            }
            this.swap(index, parentIndex);
            index = parentIndex;
        }
    }
}

// Heap with key-value pairs
class KeyValueHeap {
    constructor() {
        this.heap = [];
        this.keyToIndex = new Map();
    }
    
    insert(key, value) {
        const item = { key, value };
        this.heap.push(item);
        this.keyToIndex.set(key, this.heap.length - 1);
        this.heapifyUp();
    }
    
    updateKey(key, newValue) {
        if (!this.keyToIndex.has(key)) return;
        
        const index = this.keyToIndex.get(key);
        const oldValue = this.heap[index].value;
        this.heap[index].value = newValue;
        
        if (newValue < oldValue) {
            this.heapifyUpFrom(index);
        } else {
            this.heapifyDownFrom(index);
        }
    }
}
```

## 🔗 Related Problems

- **Kth Largest Element** - Use max heap of size k
- **Merge K Sorted Lists** - Min heap of list heads
- **Top K Frequent Elements** - Heap with frequencies
- **Find Median from Data Stream** - Two heaps (min and max)

## 💡 Key Takeaways

1. **Array Representation**: Heaps are efficiently stored in arrays
2. **Complete Binary Tree**: Always filled level by level
3. **Heap Property**: Parent-child relationship maintained
4. **Logarithmic Operations**: Insert and extract are O(log n)
5. **Building is Linear**: Can build heap from array in O(n)

Heaps are the foundation of priority queues and essential for many algorithms!
