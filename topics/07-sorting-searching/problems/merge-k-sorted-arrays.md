# Merge K Sorted Arrays

## 🎯 What Are We Trying To Solve?

Imagine you have K different sorted playlists and you want to merge them into one giant sorted playlist. You could merge them one by one, but there's a smarter way using a min-heap that tracks the "next song" from each playlist!

## 📝 The Problem

Given K sorted arrays, merge them into a single sorted array efficiently.

```
Example:
Input: [
  [1, 4, 7],
  [2, 5, 8],
  [3, 6, 9]
]
Output: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## 🧠 How to Think About This

Use a **min-heap** to always know which array has the smallest current element. It's like having K people in line, each holding their sorted deck of cards, and you always pick from whoever shows the smallest card!

### Visual Process
```
Arrays:       Min Heap (stores {value, arrayIndex, elementIndex}):
[1, 4, 7]     
[2, 5, 8]     Initial: [(1,0,0), (2,1,0), (3,2,0)]
[3, 6, 9]     
              Pop 1, add next from array 0: [(2,1,0), (3,2,0), (4,0,1)]
              Pop 2, add next from array 1: [(3,2,0), (4,0,1), (5,1,1)]
              ...
```

## 💻 The Code

```javascript
/**
 * Simple MinHeap implementation for this problem
 */
class MinHeap {
    constructor() {
        this.heap = [];
    }
    
    push(item) {
        this.heap.push(item);
        this.bubbleUp(this.heap.length - 1);
    }
    
    pop() {
        if (this.heap.length === 0) return null;
        if (this.heap.length === 1) return this.heap.pop();
        
        const min = this.heap[0];
        this.heap[0] = this.heap.pop();
        this.bubbleDown(0);
        return min;
    }
    
    bubbleUp(index) {
        while (index > 0) {
            const parentIndex = Math.floor((index - 1) / 2);
            if (this.heap[parentIndex].val <= this.heap[index].val) break;
            [this.heap[parentIndex], this.heap[index]] = [this.heap[index], this.heap[parentIndex]];
            index = parentIndex;
        }
    }
    
    bubbleDown(index) {
        while (true) {
            let minIndex = index;
            const leftChild = 2 * index + 1;
            const rightChild = 2 * index + 2;
            
            if (leftChild < this.heap.length && 
                this.heap[leftChild].val < this.heap[minIndex].val) {
                minIndex = leftChild;
            }
            
            if (rightChild < this.heap.length && 
                this.heap[rightChild].val < this.heap[minIndex].val) {
                minIndex = rightChild;
            }
            
            if (minIndex === index) break;
            
            [this.heap[minIndex], this.heap[index]] = [this.heap[index], this.heap[minIndex]];
            index = minIndex;
        }
    }
    
    get size() {
        return this.heap.length;
    }
}

/**
 * Merge k sorted arrays using min heap
 * Time: O(n log k) where n = total elements, k = number of arrays
 * Space: O(k) for the heap
 */
function mergeKSortedArrays(arrays) {
    const result = [];
    const minHeap = new MinHeap();
    
    // Initialize heap with first element from each array
    for (let i = 0; i < arrays.length; i++) {
        if (arrays[i].length > 0) {
            minHeap.push({
                val: arrays[i][0],
                arrayIndex: i,
                elementIndex: 0
            });
        }
    }
    
    // Process elements
    while (minHeap.size > 0) {
        // Get minimum element
        const { val, arrayIndex, elementIndex } = minHeap.pop();
        result.push(val);
        
        // Add next element from the same array if exists
        if (elementIndex + 1 < arrays[arrayIndex].length) {
            minHeap.push({
                val: arrays[arrayIndex][elementIndex + 1],
                arrayIndex: arrayIndex,
                elementIndex: elementIndex + 1
            });
        }
    }
    
    return result;
}

// Test cases
console.log(mergeKSortedArrays([
    [1, 4, 7],
    [2, 5, 8],
    [3, 6, 9]
])); // [1,2,3,4,5,6,7,8,9]

console.log(mergeKSortedArrays([
    [1, 3, 5],
    [2, 4, 6],
    [],
    [0, 7, 8]
])); // [0,1,2,3,4,5,6,7,8]
```

## 🎨 Step-by-Step Walkthrough

Let's merge [[1,4,7], [2,5,8], [3,6,9]]:

```
Initial State:
Arrays:        Heap:           Result:
[1,4,7]        [(1,0,0),       []
[2,5,8]         (2,1,0),
[3,6,9]         (3,2,0)]

Step 1: Pop min (1,0,0)
- Add 1 to result
- Add next from array 0: (4,0,1)
Arrays:        Heap:           Result:
[•,4,7]        [(2,1,0),       [1]
[2,5,8]         (3,2,0),
[3,6,9]         (4,0,1)]

Step 2: Pop min (2,1,0)
- Add 2 to result
- Add next from array 1: (5,1,1)
Arrays:        Heap:           Result:
[•,4,7]        [(3,2,0),       [1,2]
[•,5,8]         (4,0,1),
[3,6,9]         (5,1,1)]

Step 3: Pop min (3,2,0)
- Add 3 to result
- Add next from array 2: (6,2,1)
Arrays:        Heap:           Result:
[•,4,7]        [(4,0,1),       [1,2,3]
[•,5,8]         (5,1,1),
[•,6,9]         (6,2,1)]

Continue until heap is empty...
Final: [1,2,3,4,5,6,7,8,9]
```

## 📊 Complexity Analysis

### Time: O(n log k)
- n = total number of elements across all arrays
- Each element is pushed and popped once
- Heap operations are O(log k) where k = number of arrays

### Space: O(k)
- Heap stores at most k elements (one from each array)
- Result array is O(n) but that's the output

### Why Better Than Merging One by One?
```javascript
// Naive approach: merge arrays one by one
function mergeNaive(arrays) {
    let result = arrays[0];
    for (let i = 1; i < arrays.length; i++) {
        result = mergeTwoArrays(result, arrays[i]);
    }
    return result;
}
// Time: O(nk) - much worse!
```

## 🚫 Common Mistakes

1. **Forgetting Empty Arrays**
   ```javascript
   // Wrong: Assumes all arrays have elements
   for (let i = 0; i < arrays.length; i++) {
       heap.push(arrays[i][0]);  // Crashes on empty array!
   }
   
   // Right: Check for empty arrays
   if (arrays[i].length > 0) {
       heap.push({...});
   }
   ```

2. **Not Tracking Array Indices**
   ```javascript
   // Wrong: Only storing values in heap
   heap.push(arrays[i][0]);
   // How do we know which array to get next element from?
   
   // Right: Store value with metadata
   heap.push({
       val: arrays[i][0],
       arrayIndex: i,
       elementIndex: 0
   });
   ```

3. **Incorrect Heap Comparison**
   ```javascript
   // Wrong: Comparing objects instead of values
   if (this.heap[left] < this.heap[right])
   
   // Right: Compare the actual values
   if (this.heap[left].val < this.heap[right].val)
   ```

## 🔄 Alternative Approaches

### Approach 2: Divide and Conquer
```javascript
function mergeKSortedDivideConquer(arrays) {
    if (arrays.length === 0) return [];
    
    // Merge arrays in pairs
    while (arrays.length > 1) {
        const merged = [];
        
        // Merge pairs
        for (let i = 0; i < arrays.length; i += 2) {
            if (i + 1 < arrays.length) {
                merged.push(mergeTwoArrays(arrays[i], arrays[i + 1]));
            } else {
                merged.push(arrays[i]);
            }
        }
        
        arrays = merged;
    }
    
    return arrays[0];
}

function mergeTwoArrays(arr1, arr2) {
    const result = [];
    let i = 0, j = 0;
    
    while (i < arr1.length && j < arr2.length) {
        if (arr1[i] <= arr2[j]) {
            result.push(arr1[i++]);
        } else {
            result.push(arr2[j++]);
        }
    }
    
    while (i < arr1.length) result.push(arr1[i++]);
    while (j < arr2.length) result.push(arr2[j++]);
    
    return result;
}
// Time: O(n log k), Space: O(n)
```

### Approach 3: Using Built-in Sort (Simple but Less Efficient)
```javascript
function mergeKSortedSimple(arrays) {
    return arrays.flat().sort((a, b) => a - b);
}
// Time: O(n log n), Space: O(n)
// Less efficient but very simple!
```

## 🔗 Related Problems

- **Merge Two Sorted Arrays** - Building block for this problem
- **Find K-th Smallest Element in Sorted Matrix** - Similar heap approach
- **Merge K Sorted Lists** - Same concept with linked lists
- **Sort Nearly Sorted Array** - Related heap usage

## 💡 Key Takeaways

1. **Min-Heap is Perfect**: Always gives us the global minimum
2. **Track Metadata**: Store array and element indices with values
3. **Better Than Sequential**: O(n log k) beats O(nk)
4. **Handle Edge Cases**: Empty arrays, single array, etc.

This pattern is essential for external sorting and distributed systems!
