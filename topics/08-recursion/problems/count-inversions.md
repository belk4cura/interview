# Count Inversions

## 🎯 What Are We Trying To Solve?

Imagine you're a teacher grading tests, and you want to measure how "out of order" the students are when sorted by their test scores. Each time a student with a lower score appears before a student with a higher score, that's an "inversion". It's like counting how many swaps bubble sort would need!

## 📝 The Problem

Count the number of inversions in an array. An inversion is a pair of indices (i, j) where i < j but arr[i] > arr[j].

```
Example:
Input: [2, 4, 1, 3, 5]
Output: 3

Inversions: (2,1), (4,1), (4,3)
- Index 0 and 2: 2 > 1
- Index 1 and 2: 4 > 1
- Index 1 and 3: 4 > 3
```

## 🧠 How to Think About This

Use a modified merge sort! During the merge step, when an element from the right subarray is smaller than an element from the left subarray, we've found inversions. The key insight: if arr[i] > arr[j] where i is in the left half and j is in the right half, then ALL remaining elements in the left half are also greater than arr[j]!

### Visual Process
```
[2, 4, 1, 3, 5]

Split into:
[2, 4] | [1, 3, 5]

When merging:
- Compare 2 with 1: 2 > 1, so (2,1) and (4,1) are inversions
- Compare 4 with 3: 4 > 3, so (4,3) is an inversion

Total: 3 inversions
```

## 💻 The Code

```javascript
/**
 * Count inversions using modified merge sort
 * Time: O(n log n) - same as merge sort
 * Space: O(n) for temporary array
 */
function countInversions(arr) {
    // Create temporary array once to avoid repeated allocations
    const temp = new Array(arr.length);
    // Make a copy to preserve original array
    const arrCopy = [...arr];
    return mergeSortAndCount(arrCopy, temp, 0, arr.length - 1);
}

/**
 * Merge sort with inversion counting
 */
function mergeSortAndCount(arr, temp, left, right) {
    let invCount = 0;
    
    // Base case: single element has no inversions
    if (left < right) {
        const mid = Math.floor((left + right) / 2);
        
        // Count inversions in left half
        invCount += mergeSortAndCount(arr, temp, left, mid);
        
        // Count inversions in right half
        invCount += mergeSortAndCount(arr, temp, mid + 1, right);
        
        // Count inversions across halves during merge
        invCount += mergeAndCount(arr, temp, left, mid, right);
    }
    
    return invCount;
}

/**
 * Merge two sorted halves and count cross inversions
 */
function mergeAndCount(arr, temp, left, mid, right) {
    let i = left;      // Left subarray index
    let j = mid + 1;   // Right subarray index  
    let k = left;      // Merged array index
    let invCount = 0;
    
    // Merge while counting inversions
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) {
            // No inversion: left element ≤ right element
            temp[k++] = arr[i++];
        } else {
            // INVERSION FOUND! arr[i] > arr[j] but i < j
            temp[k++] = arr[j++];
            
            // KEY INSIGHT:
            // If arr[i] > arr[j], then ALL elements from
            // arr[i] to arr[mid] are also > arr[j]
            // Why? The left subarray is sorted!
            // So we have (mid - i + 1) inversions
            invCount += (mid - i + 1);
        }
    }
    
    // Copy remaining elements (no more inversions)
    while (i <= mid) {
        temp[k++] = arr[i++];
    }
    while (j <= right) {
        temp[k++] = arr[j++];
    }
    
    // Copy sorted elements back to original array
    for (i = left; i <= right; i++) {
        arr[i] = temp[i];
    }
    
    return invCount;
}

// Test cases
console.log(countInversions([2, 4, 1, 3, 5]));  // 3
console.log(countInversions([1, 2, 3, 4, 5]));  // 0 (already sorted)
console.log(countInversions([5, 4, 3, 2, 1]));  // 10 (reverse sorted)
console.log(countInversions([2, 3, 8, 6, 1]));  // 5
```

## 🎨 Step-by-Step Walkthrough

Let's count inversions in [2, 4, 1, 3, 5]:

```
Initial: [2, 4, 1, 3, 5]

Step 1: Divide
[2, 4] | [1, 3, 5]

Step 2: Recursively count in left half [2, 4]
[2] | [4]
No inversions within each single element
Merge: 2 < 4, no inversions
Left half total: 0

Step 3: Recursively count in right half [1, 3, 5]
[1] | [3, 5]
Further divide [3, 5] → [3] | [5]
No inversions in singles, 3 < 5 when merging
Right half total: 0

Step 4: Merge [2, 4] and [1, 3, 5] counting cross inversions
Compare 2 and 1:
- 2 > 1, so elements [2, 4] are both > 1
- Count: 2 inversions (2,1) and (4,1)
- Take 1

Compare 2 and 3:
- 2 < 3, no inversion
- Take 2

Compare 4 and 3:
- 4 > 3, so [4] is > 3
- Count: 1 inversion (4,3)
- Take 3

Remaining: 4 < 5, no inversion

Total inversions: 0 + 0 + 3 = 3
```

## 📊 Why This Works

### The Magic of Merge Sort

```
Key insight during merge of [L1, L2, L3] and [R1, R2, R3]:

If L2 > R1, then:
- (L2, R1) is an inversion
- (L3, R1) is also an inversion (since L3 ≥ L2)

So we count ALL remaining elements in left half!
```

### Complexity Analysis

```
Recurrence: T(n) = 2T(n/2) + O(n)
- Divide: O(1)
- Conquer: 2 subproblems of size n/2
- Combine: O(n) for merge

By Master Theorem: O(n log n)

This beats the naive O(n²) approach of checking all pairs!
```

## 🚫 Common Mistakes

1. **Forgetting to Count All Inversions**
   ```javascript
   // Wrong: Only counting one inversion
   if (arr[i] > arr[j]) {
       invCount++;  // Missing the other elements!
   }
   
   // Right: Count all remaining elements in left half
   if (arr[i] > arr[j]) {
       invCount += (mid - i + 1);
   }
   ```

2. **Not Using Temporary Array**
   ```javascript
   // Wrong: Modifying during comparison
   arr[k] = arr[i] < arr[j] ? arr[i++] : arr[j++];
   
   // Right: Use temp array for merging
   temp[k] = arr[i] <= arr[j] ? arr[i++] : arr[j++];
   ```

3. **Off-by-One Errors**
   ```javascript
   // Wrong: Missing the +1
   invCount += (mid - i);
   
   // Right: Include current element
   invCount += (mid - i + 1);
   ```

## 🔄 Alternative Approaches

### Approach 2: Naive O(n²)
```javascript
function countInversionsNaive(arr) {
    let count = 0;
    for (let i = 0; i < arr.length - 1; i++) {
        for (let j = i + 1; j < arr.length; j++) {
            if (arr[i] > arr[j]) {
                count++;
            }
        }
    }
    return count;
}
// Simple but slow for large arrays!
```

### Approach 3: Using Binary Indexed Tree
```javascript
class BIT {
    constructor(size) {
        this.tree = new Array(size + 1).fill(0);
    }
    
    update(idx, val) {
        while (idx < this.tree.length) {
            this.tree[idx] += val;
            idx += idx & (-idx);
        }
    }
    
    query(idx) {
        let sum = 0;
        while (idx > 0) {
            sum += this.tree[idx];
            idx -= idx & (-idx);
        }
        return sum;
    }
}

function countInversionsBIT(arr) {
    // Coordinate compression
    const sorted = [...new Set(arr)].sort((a, b) => a - b);
    const rank = new Map();
    sorted.forEach((val, idx) => rank.set(val, idx + 1));
    
    const bit = new BIT(sorted.length);
    let inversions = 0;
    
    for (let i = arr.length - 1; i >= 0; i--) {
        const r = rank.get(arr[i]);
        inversions += bit.query(r - 1);
        bit.update(r, 1);
    }
    
    return inversions;
}
// Time: O(n log n), but more complex to implement
```

## 🔗 Related Problems

- **Merge Sort** - The foundation of this algorithm
- **Reverse Pairs** - Count pairs where arr[i] > 2 * arr[j]
- **Count Smaller Numbers After Self** - Similar concept
- **Global and Local Inversions** - Variation with constraints

## 💡 Key Takeaways

1. **Divide and Conquer Insight**: Count inversions while merging
2. **Batch Counting**: When one element is inverted, count all related inversions
3. **Preserves Sorting**: Get both sorted array and inversion count
4. **Optimal Complexity**: O(n log n) beats naive O(n²)

This problem beautifully demonstrates how modifying a classic algorithm can solve a different problem!
