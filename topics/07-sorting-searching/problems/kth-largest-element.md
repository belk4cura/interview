# Find Kth Largest Element

## 🎯 What Are We Trying To Solve?

Imagine you're a teacher who needs to find the student with the 3rd highest score, but you don't want to sort all 1000 test papers. You want a quick way to find just that one score without organizing everything. That's what we're doing here - finding the Kth largest element efficiently!

## 📝 The Problem

Find the kth largest element in an unsorted array. Note that it is the kth largest element in sorted order, not the kth distinct element.

```
Examples:
Input: [3,2,1,5,6,4], k = 2
Output: 5 (The 2nd largest is 5)

Input: [3,2,3,1,2,4,5,5,6], k = 4
Output: 4 (The 4th largest is 4)
```

## 🧠 How to Think About This

We can use **QuickSelect** - a variation of QuickSort that only recurses into the side containing our target. It's like binary search but for unsorted arrays!

### Visual Process
```
Finding 2nd largest in [3,2,1,5,6,4]:

After partition around pivot 4:
[3,2,1] 4 [5,6]
   ↑        ↑
 smaller  larger

We need 2nd largest → look in right side
Continue with [5,6]...
```

## 💻 The Code

```javascript
/**
 * Find kth largest using QuickSelect
 * Time: O(n) average, O(n²) worst case
 * Space: O(1) in-place
 */
function findKthLargest(nums, k) {
    // Convert "kth largest" to "index in sorted array"
    // 1st largest = index n-1, 2nd largest = index n-2, etc.
    const targetIndex = nums.length - k;
    
    return quickSelect(nums, 0, nums.length - 1, targetIndex);
}

/**
 * QuickSelect algorithm - finds element at target index
 */
function quickSelect(nums, left, right, targetIndex) {
    // Base case: single element
    if (left === right) {
        return nums[left];
    }
    
    // Partition array and get pivot's final position
    const pivotIndex = partition(nums, left, right);
    
    if (pivotIndex === targetIndex) {
        // Found it! Pivot is at the target position
        return nums[pivotIndex];
    } else if (targetIndex < pivotIndex) {
        // Target is in left partition
        return quickSelect(nums, left, pivotIndex - 1, targetIndex);
    } else {
        // Target is in right partition
        return quickSelect(nums, pivotIndex + 1, right, targetIndex);
    }
}

/**
 * Partition array around a pivot
 * Returns final position of pivot
 */
function partition(nums, left, right) {
    // Randomize pivot to avoid worst case
    const randomIndex = Math.floor(Math.random() * (right - left + 1)) + left;
    swap(nums, randomIndex, right);
    
    const pivot = nums[right];
    let i = left - 1;  // Boundary of smaller elements
    
    // Move smaller elements to left
    for (let j = left; j < right; j++) {
        if (nums[j] < pivot) {
            i++;
            swap(nums, i, j);
        }
    }
    
    // Place pivot in correct position
    swap(nums, i + 1, right);
    return i + 1;
}

function swap(nums, i, j) {
    [nums[i], nums[j]] = [nums[j], nums[i]];
}

// Test cases
console.log(findKthLargest([3,2,1,5,6,4], 2));           // 5
console.log(findKthLargest([3,2,3,1,2,4,5,5,6], 4));     // 4
console.log(findKthLargest([1], 1));                     // 1
```

## 🎨 Step-by-Step Walkthrough

Finding 2nd largest in [3,2,1,5,6,4]:

```
k = 2, so targetIndex = 6 - 2 = 4

Step 1: Partition with pivot 4
[3,2,1] 4 [5,6]
        ↑
    index 3

Step 2: Compare pivotIndex (3) with targetIndex (4)
- 3 < 4, so search right partition [5,6]
- New range: left=4, right=5

Step 3: Partition [5,6] with pivot 6
[5] 6
    ↑
index 5

Step 4: Compare pivotIndex (5) with targetIndex (4)
- 5 > 4, so search left partition [5]
- New range: left=4, right=4

Step 5: Single element at index 4
Return nums[4] = 5 ✓
```

## 📊 Why Average O(n)?

### Best Case Analysis
```
Each partition roughly halves the problem:
n + n/2 + n/4 + n/8 + ... = 2n = O(n)

Unlike QuickSort, we only recurse on ONE side!
```

### Worst Case O(n²)
```
If we always pick the smallest/largest as pivot:
n + (n-1) + (n-2) + ... + 1 = n²/2 = O(n²)

Solution: Random pivot selection makes this unlikely!
```

## 🚫 Common Mistakes

1. **Wrong Index Conversion**
   ```javascript
   // Wrong: Forgetting to convert k to index
   return quickSelect(nums, 0, nums.length - 1, k);
   
   // Right: kth largest → (n-k)th smallest
   const targetIndex = nums.length - k;
   ```

2. **Not Handling Duplicates**
   ```javascript
   // This algorithm handles duplicates naturally!
   // [3,3,3,3], k=2 correctly returns 3
   ```

3. **Modifying Original Array**
   ```javascript
   // If you need to preserve original:
   function findKthLargest(nums, k) {
       const copy = [...nums];  // Work on copy
       return quickSelect(copy, 0, copy.length - 1, nums.length - k);
   }
   ```

## 🔄 Alternative Approaches

### Approach 2: Using Min Heap (Size K)
```javascript
function findKthLargestHeap(nums, k) {
    // Min heap to maintain k largest elements
    const minHeap = new MinHeap();
    
    for (const num of nums) {
        minHeap.add(num);
        
        // Keep only k largest
        if (minHeap.size() > k) {
            minHeap.poll();  // Remove smallest
        }
    }
    
    // Root of min heap is kth largest
    return minHeap.peek();
}
// Time: O(n log k), Space: O(k)
```

### Approach 3: Sorting (Simple but Less Efficient)
```javascript
function findKthLargestSort(nums, k) {
    nums.sort((a, b) => b - a);  // Sort descending
    return nums[k - 1];
}
// Time: O(n log n), Space: O(1) or O(n) depending on sort
```

### Approach 4: Counting Sort (For Limited Range)
```javascript
function findKthLargestCounting(nums, k) {
    const min = Math.min(...nums);
    const max = Math.max(...nums);
    const count = new Array(max - min + 1).fill(0);
    
    // Count occurrences
    for (const num of nums) {
        count[num - min]++;
    }
    
    // Find kth largest
    let remaining = k;
    for (let i = count.length - 1; i >= 0; i--) {
        remaining -= count[i];
        if (remaining <= 0) {
            return i + min;
        }
    }
}
// Time: O(n + range), Space: O(range)
```

## 🔗 Related Problems

- **Kth Smallest Element** - Same approach, different index
- **Top K Frequent Elements** - Combine with frequency counting
- **Find Median** - Special case where k = n/2
- **K Closest Points to Origin** - QuickSelect with distance

## 💡 Key Takeaways

1. **QuickSelect is Efficient**: Average O(n) beats sorting's O(n log n)
2. **Random Pivot is Crucial**: Avoids worst-case behavior
3. **In-Place Algorithm**: Space efficient O(1)
4. **Partial Sorting**: We only sort as much as needed

This is a must-know algorithm for technical interviews!
