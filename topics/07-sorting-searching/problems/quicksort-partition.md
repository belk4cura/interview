# QuickSort and Partition

## 🎯 What Are We Trying To Solve?

QuickSort is a divide-and-conquer algorithm that picks a "pivot" element and partitions the array around it - all smaller elements go to the left, all larger elements go to the right. It's like organizing a group photo by height, with one person as the reference point!

## 📝 The Problem

Implement the partition function for QuickSort, which is the key operation that rearranges the array around a pivot element. Then use it to build the complete QuickSort algorithm.

```
Example Partition:
Array: [4, 5, 3, 7, 2]
Pivot: 4 (first element)

After partition:
Left: [3, 2]  (elements < 4)
Pivot: 4
Right: [5, 7] (elements ≥ 4)

Result: [3, 2, 4, 5, 7]
```

## 💻 The Code

```javascript
/**
 * Partition array around pivot (first element)
 * @param {number[]} arr - Array to partition
 * @returns {number[][]} - [left, pivot, right] arrays
 */
function partition(arr) {
    if (arr.length <= 1) return [[], arr[0], []];
    
    const pivot = arr[0];
    const left = [];
    const right = [];
    
    // Partition elements (skip pivot at index 0)
    for (let i = 1; i < arr.length; i++) {
        if (arr[i] < pivot) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }
    
    return [left, pivot, right];
}

/**
 * QuickSort - complete implementation
 * @param {number[]} arr - Array to sort
 * @returns {number[]} - New sorted array
 */
function quickSort(arr) {
    // Base case: arrays with 0 or 1 element are sorted
    if (arr.length <= 1) return arr;
    
    // Choose pivot (first element)
    const pivot = arr[0];
    const left = [];
    const right = [];
    
    // Partition
    for (let i = 1; i < arr.length; i++) {
        if (arr[i] < pivot) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }
    
    // Recursively sort and combine
    return [...quickSort(left), pivot, ...quickSort(right)];
}

/**
 * In-place partition (Lomuto scheme)
 * @param {number[]} arr - Array to partition
 * @param {number} low - Start index
 * @param {number} high - End index (pivot)
 * @returns {number} - Final position of pivot
 */
function partitionInPlace(arr, low, high) {
    // Choose last element as pivot
    const pivot = arr[high];
    
    // Index of smaller element
    let i = low - 1;
    
    for (let j = low; j < high; j++) {
        // If current element is smaller than pivot
        if (arr[j] < pivot) {
            i++;
            // Swap arr[i] and arr[j]
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
    }
    
    // Place pivot in correct position
    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
    
    return i + 1;  // Return pivot's final position
}

/**
 * In-place QuickSort
 * @param {number[]} arr - Array to sort
 * @param {number} low - Start index
 * @param {number} high - End index
 * @returns {number[]} - Same array, sorted
 */
function quickSortInPlace(arr, low = 0, high = arr.length - 1) {
    if (low < high) {
        // Find pivot position after partition
        const pivotIndex = partitionInPlace(arr, low, high);
        
        // Recursively sort left and right
        quickSortInPlace(arr, low, pivotIndex - 1);
        quickSortInPlace(arr, pivotIndex + 1, high);
    }
    
    return arr;
}

/**
 * Alternative: Hoare partition scheme
 * More efficient than Lomuto
 */
function partitionHoare(arr, low, high) {
    const pivot = arr[Math.floor((low + high) / 2)];
    let i = low - 1;
    let j = high + 1;
    
    while (true) {
        // Find element from left >= pivot
        do {
            i++;
        } while (arr[i] < pivot);
        
        // Find element from right <= pivot
        do {
            j--;
        } while (arr[j] > pivot);
        
        // If pointers crossed, partitioning is done
        if (i >= j) {
            return j;
        }
        
        // Swap elements
        [arr[i], arr[j]] = [arr[j], arr[i]];
    }
}

/**
 * 3-way partition for duplicates
 * Useful when array has many duplicate values
 */
function partition3Way(arr, low, high) {
    if (high <= low) return;
    
    let lt = low;      // arr[low..lt-1] < pivot
    let gt = high;     // arr[gt+1..high] > pivot
    let i = low + 1;   // arr[lt..i-1] == pivot
    const pivot = arr[low];
    
    while (i <= gt) {
        if (arr[i] < pivot) {
            [arr[lt], arr[i]] = [arr[i], arr[lt]];
            lt++;
            i++;
        } else if (arr[i] > pivot) {
            [arr[i], arr[gt]] = [arr[gt], arr[i]];
            gt--;
        } else {
            i++;
        }
    }
    
    return [lt, gt];
}

// Test cases
console.log(quickSort([4, 5, 3, 7, 2]));  // [2, 3, 4, 5, 7]
console.log(quickSort([3, 1, 4, 1, 5, 9, 2, 6]));  // [1, 1, 2, 3, 4, 5, 6, 9]

const arr1 = [4, 5, 3, 7, 2];
quickSortInPlace(arr1);
console.log(arr1);  // [2, 3, 4, 5, 7]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `quickSort([4, 5, 3, 7, 2])`:

```
Initial: [4, 5, 3, 7, 2]

Step 1: Partition around pivot=4
- Left (< 4): [3, 2]
- Pivot: 4
- Right (≥ 4): [5, 7]

Step 2: Recursively sort left [3, 2]
- Pivot=3, Left=[2], Right=[]
- Recurse: [2] and [] are base cases
- Result: [2, 3]

Step 3: Recursively sort right [5, 7]
- Pivot=5, Left=[], Right=[7]
- Recurse: [] and [7] are base cases
- Result: [5, 7]

Step 4: Combine
[2, 3] + 4 + [5, 7] = [2, 3, 4, 5, 7]
```

## 📊 Visual Understanding

```
QuickSort Process:

       [4, 5, 3, 7, 2]
            ↓
    Partition (pivot=4)
       /    |    \
    [3,2]  [4]  [5,7]
      ↓           ↓
   Sort        Sort
    [2,3]      [5,7]
      ↓           ↓
    Combine all parts
    [2, 3, 4, 5, 7]

In-Place Partition (Lomuto):

Initial: [4, 5, 3, 7, 2]  pivot=2 (last)
         i=-1

j=0: 4>2, no swap, i=-1
j=1: 5>2, no swap, i=-1
j=2: 3>2, no swap, i=-1
j=3: 7>2, no swap, i=-1

Final swap: arr[i+1]=arr[0] with pivot
Result: [2, 5, 3, 7, 4]
         ↑ pivot in position
```

## ⏱️ Complexity Analysis

- **Time Complexity:**
  - Best Case: O(n log n) - Balanced partitions
  - Average Case: O(n log n) - Random data
  - Worst Case: O(n²) - Already sorted array (poor pivot choice)
  
- **Space Complexity:**
  - Out-of-place: O(n) - New arrays created
  - In-place: O(log n) - Recursion stack depth

## 🚫 Common Mistakes

1. **Off-by-one in partition boundaries**
   ```javascript
   // Wrong: Including pivot in recursion
   quickSort(arr, low, pivotIndex);  // Should exclude pivot
   
   // Right: Exclude pivot from recursive calls
   quickSort(arr, low, pivotIndex - 1);
   quickSort(arr, pivotIndex + 1, high);
   ```

2. **Not handling empty arrays**
   ```javascript
   // Wrong: Crashes on empty array
   const pivot = arr[0];  // arr[0] is undefined!
   
   // Right: Check array length first
   if (arr.length <= 1) return arr;
   ```

3. **Modifying array while iterating**
   ```javascript
   // Wrong: Splice during iteration
   for (let i = 0; i < arr.length; i++) {
       if (arr[i] < pivot) {
           arr.splice(i, 1);  // Changes indices!
       }
   }
   
   // Right: Build new arrays
   const left = [];
   const right = [];
   ```

## 🔄 Variations

```javascript
// Random pivot selection (improves average case)
function quickSortRandom(arr) {
    if (arr.length <= 1) return arr;
    
    // Random pivot
    const pivotIndex = Math.floor(Math.random() * arr.length);
    const pivot = arr[pivotIndex];
    
    const left = [];
    const right = [];
    
    for (let i = 0; i < arr.length; i++) {
        if (i === pivotIndex) continue;
        if (arr[i] < pivot) {
            left.push(arr[i]);
        } else {
            right.push(arr[i]);
        }
    }
    
    return [...quickSortRandom(left), pivot, ...quickSortRandom(right)];
}

// Find kth smallest element (QuickSelect)
function quickSelect(arr, k, low = 0, high = arr.length - 1) {
    if (low === high) return arr[low];
    
    const pivotIndex = partitionInPlace(arr, low, high);
    
    if (k === pivotIndex) {
        return arr[k];
    } else if (k < pivotIndex) {
        return quickSelect(arr, k, low, pivotIndex - 1);
    } else {
        return quickSelect(arr, k, pivotIndex + 1, high);
    }
}
```

## 🔗 Related Problems

- **Merge Sort** - Another divide-and-conquer sort
- **Kth Largest Element** - Uses partition (QuickSelect)
- **Sort Colors** - 3-way partitioning
- **Nuts and Bolts** - Partition matching problem

## 💡 Key Takeaways

1. **Partition is Key**: The core operation of QuickSort
2. **Pivot Choice Matters**: Random pivot avoids worst case
3. **In-Place vs Out-of-Place**: Trade-off between space and simplicity
4. **Not Stable**: Doesn't preserve relative order of equal elements
5. **Cache Friendly**: Good locality of reference

QuickSort is often the fastest practical sorting algorithm due to its cache efficiency!
