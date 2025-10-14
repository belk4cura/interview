# Space Complexity

## 🎯 What Are We Trying To Solve?

Space complexity is about memory usage. Imagine your computer's memory is like a backpack - how much stuff do you need to carry to solve the problem? Can you travel light (O(1)) or do you need a moving truck (O(n²))?

## 📝 The Problem

Analyze the space complexity of this merge sort implementation:

```javascript
function mystery2(arr) {
    // Base case: arrays with 0 or 1 element are already "sorted"
    if (arr.length <= 1) return arr;
    
    // arr.slice(start, end) creates a NEW array (shallow copy) from index start to end (exclusive)
    // This splits the array in half - takes elements from index 0 up to the middle
    // Example: [1,2,3,4,5].slice(0, 2) → [1,2] (first half)
    const left = mystery2(arr.slice(0, Math.floor(arr.length / 2)));
    
    // arr.slice(start) with no end parameter goes to the end of the array
    // This gets the second half - from middle to end
    // Example: [1,2,3,4,5].slice(2) → [3,4,5] (second half)
    const right = mystery2(arr.slice(Math.floor(arr.length / 2)));
    
    // Spread operator [...left, ...right] creates ANOTHER new array by combining both halves
    // This is where merge sort would normally merge/sort, but this just concatenates
    return [...left, ...right];
}
```

## 🧠 How to Think About Space

Let's visualize what happens in memory:

### The Call Stack Tree
```
Original: [8, 3, 5, 4, 7, 6, 1, 2]

Level 0:        [8,3,5,4,7,6,1,2]
                /                \
Level 1:   [8,3,5,4]          [7,6,1,2]
           /        \          /        \
Level 2: [8,3]     [5,4]    [7,6]     [1,2]
         /   \     /   \    /   \     /   \
Level 3:[8]  [3]  [5]  [4] [7]  [6]  [1]  [2]
```

### Memory at Each Level
```
Level 0: 8 elements stored
Level 1: 4 + 4 = 8 elements stored
Level 2: 2 + 2 + 2 + 2 = 8 elements stored
Level 3: 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 = 8 elements

Total new arrays created: 8 × log(8) = 8 × 3 = 24 elements!
```

## 💡 The Answer

**Space Complexity: O(n log n)**

### Why O(n log n)?
1. **Each level creates arrays totaling n elements**: O(n) per level
2. **Tree has log n levels**: The recursion depth
3. **Total space**: O(n) × O(log n) = O(n log n)

### Visual Memory Usage
```
Memory Boxes (n = 8):

Original:  [📦][📦][📦][📦][📦][📦][📦][📦]

During recursion (all exist simultaneously):
Level 0:   [📦][📦][📦][📦][📦][📦][📦][📦]
Level 1:   [📦][📦][📦][📦] [📦][📦][📦][📦]
Level 2:   [📦][📦] [📦][📦] [📦][📦] [📦][📦]
Level 3:   [📦] [📦] [📦] [📦] [📦] [📦] [📦] [📦]

Total boxes: 32 (way more than original 8!)
```

## 📊 Breaking Down Space Usage

### 1. The Arrays (Main Space User)
```javascript
arr.slice(0, Math.floor(arr.length / 2))  // Creates NEW array
arr.slice(Math.floor(arr.length / 2))     // Creates ANOTHER new array
[...left, ...right]                       // Creates YET ANOTHER array
```

### 2. The Call Stack (Additional Space)
```
Each recursive call adds a frame to the stack:
┌─────────────────┐
│ mystery2([8])   │ ← Level 3
├─────────────────┤
│ mystery2([8,3]) │ ← Level 2
├─────────────────┤
│mystery2([8,3,5,4])│ ← Level 1
├─────────────────┤
│mystery2([8,3,5,4,7,6,1,2])│ ← Level 0
└─────────────────┘

Stack depth = log n frames
```

## 🎨 Step-by-Step Memory Trace

Let's trace with a small example: [4, 2, 3, 1]

```javascript
mystery2([4, 2, 3, 1])
  Memory: [4, 2, 3, 1]                    // 4 elements
  
  ├─ mystery2([4, 2])
  │   Memory: [4, 2]                      // +2 elements
  │   
  │   ├─ mystery2([4])
  │   │   Memory: [4]                     // +1 element
  │   │   Returns: [4]
  │   │
  │   └─ mystery2([2])
  │       Memory: [2]                     // +1 element
  │       Returns: [2]
  │   
  │   Returns: [4, 2]                     // +2 elements (merge)
  │
  └─ mystery2([3, 1])
      Memory: [3, 1]                      // +2 elements
      
      ├─ mystery2([3])
      │   Memory: [3]                     // +1 element
      │   Returns: [3]
      │
      └─ mystery2([1])
          Memory: [1]                     // +1 element
          Returns: [1]
      
      Returns: [3, 1]                     // +2 elements (merge)
  
  Final: [4, 2, 3, 1]                    // +4 elements (final merge)

Total memory used: 4 + 2 + 2 + 1 + 1 + 1 + 1 + 2 + 2 + 4 = 20 elements
For n=4: 20 elements ≈ n × log n = 4 × 2 = 8 (with constants)
```

## 🚫 Common Mistakes

1. **Forgetting About Slice Creating New Arrays**
   ```javascript
   // This creates a NEW array, not a view!
   arr.slice(0, mid)  // O(n/2) space
   ```

2. **Not Counting All Levels**
   - Each recursion level exists simultaneously
   - Must count space across entire tree

3. **Ignoring the Call Stack**
   - Each function call takes stack space
   - Depth of recursion = O(log n) additional space

## 🔄 Compare: Space-Efficient Version

```javascript
// In-place merge sort (O(1) extra space, harder to implement)
function mergeSortInPlace(arr, left, right) {
    if (left >= right) return;
    
    const mid = Math.floor((left + right) / 2);
    mergeSortInPlace(arr, left, mid);
    mergeSortInPlace(arr, mid + 1, right);
    mergeInPlace(arr, left, mid, right);  // No new arrays!
}

// Space: O(log n) - Only the call stack
```

## 📈 Space Complexity Comparison

```
Algorithm        Space Used for n=1000
─────────────────────────────────────
Selection Sort   O(1)      = 1 variable
Merge Sort       O(n)      = 1,000 elements  
This Version     O(n log n) = 10,000 elements!
Matrix Storage   O(n²)     = 1,000,000 elements
```

## 💾 Real-World Impact

```javascript
// If each element is 8 bytes:
n = 1,000:     80 KB (manageable)
n = 1,000,000: 160 MB (significant!)
n = 1 billion: 240 GB (won't fit in RAM!)

// This is why databases use external merge sort for large datasets
```

## 🔗 Related Concepts

- Merge Sort optimization
- Quick Sort space complexity (O(log n) average)
- Iterative vs Recursive implementations
- Tail recursion optimization

## 💡 Key Takeaway

**Space matters as much as time!** This implementation trades space for simplicity. The `.slice()` operations create new arrays at every level, leading to O(n log n) space instead of the optimal O(n).

Remember:
- `.slice()` = new array = more memory
- Recursion depth = stack space
- Total space = arrays + stack
- In-place algorithms save memory but are harder to write
