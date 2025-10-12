# Insertion Sort - Correctness and Loop Invariant

## 🎯 What Are We Trying To Solve?

Insertion sort builds a sorted array one element at a time by repeatedly inserting the next element into its correct position. It's like sorting playing cards in your hand - you pick up cards one by one and insert each into its proper place among the cards you're already holding.

## 📝 The Problem

Implement insertion sort and understand its loop invariant - the property that remains true throughout the algorithm's execution. At each iteration, the elements before the current position are already sorted.

```
Example:
Input: [5, 2, 4, 6, 1, 3]

Process:
[5] 2 4 6 1 3     → Start with first element
[2, 5] 4 6 1 3    → Insert 2 in correct position
[2, 4, 5] 6 1 3   → Insert 4 in correct position
[2, 4, 5, 6] 1 3  → Insert 6 in correct position
[1, 2, 4, 5, 6] 3 → Insert 1 in correct position
[1, 2, 3, 4, 5, 6] → Insert 3 in correct position
```

## 💻 The Code

```javascript
/**
 * Insertion Sort - sorts array in place
 * @param {number[]} arr - Array to sort
 * @returns {number[]} - Sorted array (same reference)
 */
function insertionSort(arr) {
    // Start from second element (index 1)
    for (let i = 1; i < arr.length; i++) {
        // Store current element to insert
        const key = arr[i];
        
        // Start comparing with elements before it
        let j = i - 1;
        
        // Shift elements right while they're greater than key
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];  // Shift element right
            j--;                   // Move to previous element
        }
        
        // Insert key at correct position
        arr[j + 1] = key;
    }
    
    return arr;
}

// Alternative: With detailed loop invariant tracking
function insertionSortWithInvariant(arr) {
    console.log('Initial:', arr.join(' '));
    
    for (let i = 1; i < arr.length; i++) {
        // Loop Invariant: arr[0...i-1] is sorted
        console.log(`\nBefore iteration ${i}:`);
        console.log(`Sorted: [${arr.slice(0, i).join(', ')}]`);
        console.log(`To process: [${arr.slice(i).join(', ')}]`);
        
        const key = arr[i];
        let j = i - 1;
        
        // Find insertion position
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        
        arr[j + 1] = key;
        
        console.log(`After inserting ${key}: ${arr.join(' ')}`);
        // Loop Invariant maintained: arr[0...i] is now sorted
    }
    
    return arr;
}

// Alternative: Recursive insertion sort
function insertionSortRecursive(arr, n = arr.length) {
    // Base case: single element is already sorted
    if (n <= 1) return arr;
    
    // Sort first n-1 elements
    insertionSortRecursive(arr, n - 1);
    
    // Insert last element at correct position
    const last = arr[n - 1];
    let j = n - 2;
    
    while (j >= 0 && arr[j] > last) {
        arr[j + 1] = arr[j];
        j--;
    }
    
    arr[j + 1] = last;
    return arr;
}

// Alternative: Binary insertion sort (uses binary search)
function binaryInsertionSort(arr) {
    for (let i = 1; i < arr.length; i++) {
        const key = arr[i];
        const insertPos = binarySearch(arr, key, 0, i);
        
        // Shift elements and insert
        for (let j = i - 1; j >= insertPos; j--) {
            arr[j + 1] = arr[j];
        }
        arr[insertPos] = key;
    }
    return arr;
}

function binarySearch(arr, key, start, end) {
    if (start >= end) {
        return (arr[start] > key) ? start : start + 1;
    }
    
    const mid = Math.floor((start + end) / 2);
    
    if (arr[mid] === key) {
        return mid + 1;
    } else if (arr[mid] > key) {
        return binarySearch(arr, key, start, mid);
    } else {
        return binarySearch(arr, key, mid + 1, end);
    }
}

// Test cases
console.log(insertionSort([5, 2, 4, 6, 1, 3]));  // [1, 2, 3, 4, 5, 6]
console.log(insertionSort([31, 41, 59, 26, 41, 58]));  // [26, 31, 41, 41, 58, 59]
console.log(insertionSort([1]));  // [1]
console.log(insertionSort([3, 2, 1]));  // [1, 2, 3]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `insertionSort([5, 2, 4, 6, 1, 3])`:

```
Initial: [5, 2, 4, 6, 1, 3]

Iteration 1 (i=1, key=2):
- Compare 2 with 5: 5 > 2, shift 5 right
- Array: [5, 5, 4, 6, 1, 3]
- Insert 2 at position 0
- Result: [2, 5, 4, 6, 1, 3]

Iteration 2 (i=2, key=4):
- Compare 4 with 5: 5 > 4, shift 5 right
- Array: [2, 5, 5, 6, 1, 3]
- Compare 4 with 2: 2 < 4, stop
- Insert 4 at position 1
- Result: [2, 4, 5, 6, 1, 3]

Iteration 3 (i=3, key=6):
- Compare 6 with 5: 5 < 6, no shift needed
- Insert 6 at position 3 (stays in place)
- Result: [2, 4, 5, 6, 1, 3]

Iteration 4 (i=4, key=1):
- Compare 1 with 6: shift 6 right
- Compare 1 with 5: shift 5 right
- Compare 1 with 4: shift 4 right
- Compare 1 with 2: shift 2 right
- Insert 1 at position 0
- Result: [1, 2, 4, 5, 6, 3]

Iteration 5 (i=5, key=3):
- Compare 3 with 6: shift 6 right
- Compare 3 with 5: shift 5 right
- Compare 3 with 4: shift 4 right
- Compare 3 with 2: 2 < 3, stop
- Insert 3 at position 2
- Final: [1, 2, 3, 4, 5, 6]
```

## 📊 Loop Invariant

The **loop invariant** is a condition that remains true before and after each iteration:

```
Invariant: At the start of iteration i, 
           the subarray arr[0...i-1] is sorted

Proof by Induction:

Initialization (i=1):
- Subarray arr[0...0] has one element
- A single element is always sorted ✓

Maintenance:
- Assume arr[0...i-1] is sorted
- We insert arr[i] into correct position
- After insertion, arr[0...i] is sorted ✓

Termination (i=n):
- Loop ends when i = arr.length
- By invariant, arr[0...n-1] is sorted
- This is the entire array ✓
```

## ⏱️ Complexity Analysis

- **Time Complexity:** 
  - Best Case: O(n) - Already sorted array
  - Average Case: O(n²) - Random array
  - Worst Case: O(n²) - Reverse sorted array
  
- **Space Complexity:** O(1) - Sorts in place

## 🚫 Common Mistakes

1. **Starting from index 0 instead of 1**
   ```javascript
   // Wrong: First element has nothing to compare with
   for (let i = 0; i < arr.length; i++) {
   
   // Right: Start from second element
   for (let i = 1; i < arr.length; i++) {
   ```

2. **Incorrect comparison in while loop**
   ```javascript
   // Wrong: Doesn't handle j becoming -1
   while (arr[j] > key) {
   
   // Right: Check j >= 0 first
   while (j >= 0 && arr[j] > key) {
   ```

3. **Forgetting to decrement j**
   ```javascript
   // Wrong: Infinite loop
   while (j >= 0 && arr[j] > key) {
       arr[j + 1] = arr[j];
       // Missing j--
   }
   
   // Right: Decrement j
   while (j >= 0 && arr[j] > key) {
       arr[j + 1] = arr[j];
       j--;
   }
   ```

## 🔄 Variations

```javascript
// Insertion sort for linked list
class ListNode {
    constructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

function insertionSortList(head) {
    const dummy = new ListNode(0);
    let current = head;
    
    while (current) {
        const next = current.next;
        
        // Find insertion position
        let prev = dummy;
        while (prev.next && prev.next.val < current.val) {
            prev = prev.next;
        }
        
        // Insert current node
        current.next = prev.next;
        prev.next = current;
        
        current = next;
    }
    
    return dummy.next;
}

// Insertion sort with custom comparator
function insertionSortWithComparator(arr, compareFn) {
    for (let i = 1; i < arr.length; i++) {
        const key = arr[i];
        let j = i - 1;
        
        while (j >= 0 && compareFn(arr[j], key) > 0) {
            arr[j + 1] = arr[j];
            j--;
        }
        
        arr[j + 1] = key;
    }
    return arr;
}

// Usage
const people = [{name: 'Alice', age: 30}, {name: 'Bob', age: 25}];
insertionSortWithComparator(people, (a, b) => a.age - b.age);
```

## 🔗 Related Problems

- **Selection Sort** - Another O(n²) sorting algorithm
- **Bubble Sort** - Similar complexity, different approach
- **Merge Sort** - Divide and conquer approach
- **Quick Sort** - More efficient average case

## 💡 Key Takeaways

1. **Loop Invariant**: Key to proving correctness
2. **Stable Sort**: Maintains relative order of equal elements
3. **Adaptive**: Performs well on nearly sorted data
4. **In-Place**: Requires only O(1) extra space
5. **Online Algorithm**: Can sort a list as it receives it

Insertion sort is excellent for small datasets and nearly sorted arrays!
