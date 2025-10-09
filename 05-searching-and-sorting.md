# Searching and Sorting

## Overview
Searching and sorting algorithms form the foundation of efficient data processing. Understanding their complexities and trade-offs is crucial for optimizing database queries, file systems, and distributed data processing.

### Key Concepts
- **Binary Search**: O(log n) search in sorted data
- **Sorting Algorithms**: Time/space trade-offs
- **Stability**: Maintaining relative order of equal elements
- **AWS Applications**: CloudWatch Logs Insights, Athena queries, DynamoDB queries

## Searching Algorithms

### Linear Search

```javascript
// ============= LINEAR SEARCH =============
// Time: O(n) - WHY?
// Must check each element sequentially
// Worst case: element is last or not present = n checks
// Average case: element in middle = n/2 checks = O(n)
// Space: O(1) - only using loop counter
function linearSearch(arr, target) {
    // Iterate through every element
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] === target) {
            return i;  // Found at index i
        }
    }
    return -1;  // Not found after checking all n elements
}

// ============= OPTIMIZED FOR SORTED ARRAYS =============
// Still O(n) worst case, but better average case
// Can terminate early if element can't exist
function linearSearchSorted(arr, target) {
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] === target) {
            return i;
        }
        // KEY OPTIMIZATION: Early termination
        // If current element > target, target can't exist later
        // WHY? Array is sorted, all later elements are even larger
        if (arr[i] > target) {
            return -1;  // Can stop searching
        }
    }
    return -1;
}
```

### Binary Search

```javascript
// ============= BINARY SEARCH (ITERATIVE) =============
// Time: O(log n) - WHY?
// Each iteration eliminates half the search space
// n → n/2 → n/4 → n/8 → ... → 1
// Number of steps: log₂(n)
// Example: 1000 elements → ~10 comparisons (2^10 = 1024)
// Space: O(1) - only using pointers
// PREREQUISITE: Array MUST be sorted!
function binarySearch(arr, target) {
    let left = 0;
    let right = arr.length - 1;
    
    // Continue while search space is valid
    while (left <= right) {
        // Find middle point
        // Math.floor prevents decimal indices
        const mid = Math.floor((left + right) / 2);
        
        // Found target
        if (arr[mid] === target) {
            return mid;
        }
        
        // Target in right half
        // WHY mid + 1? We already checked mid
        if (arr[mid] < target) {
            left = mid + 1;  // Eliminate left half
        } else {
            right = mid - 1; // Eliminate right half
        }
    }
    
    return -1;  // Target not found
}

// ============= BINARY SEARCH (RECURSIVE) =============
// Time: O(log n) - same reasoning
// Space: O(log n) - WHY?
// Each recursive call adds stack frame
// Maximum recursion depth = log n
// Each frame uses O(1) space
function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
    // Base case: invalid search space
    if (left > right) {
        return -1;
    }
    
    const mid = Math.floor((left + right) / 2);
    
    // Base case: found target
    if (arr[mid] === target) {
        return mid;
    }
    
    // Recursive cases: search appropriate half
    if (arr[mid] < target) {
        // Search right half
        return binarySearchRecursive(arr, target, mid + 1, right);
    }
    
    // Search left half
    return binarySearchRecursive(arr, target, left, mid - 1);
}

// WHY ITERATIVE VS RECURSIVE?
// Iterative: Better space complexity O(1), no stack overflow risk
// Recursive: Cleaner code, easier to understand
// Production: Usually prefer iterative

// ============= FIND FIRST/LAST OCCURRENCE =============
// Time: O(log n) - binary search with continuation
// Space: O(1) - only using pointers
// KEY INSIGHT: Don't stop when found - continue searching
// for leftmost/rightmost occurrence
function findFirst(arr, target) {
    let left = 0;
    let right = arr.length - 1;
    let result = -1;  // Store first occurrence found
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) {
            result = mid;  // Save this occurrence
            // KEY: Continue searching LEFT for earlier occurrence
            // WHY? There might be duplicates to the left
            right = mid - 1;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}

function findLast(arr, target) {
    let left = 0;
    let right = arr.length - 1;
    let result = -1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) {
            result = mid;  // Save this occurrence
            // KEY: Continue searching RIGHT for later occurrence
            // WHY? There might be duplicates to the right
            left = mid + 1;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}
```

### Binary Search Variations

```javascript
// ============= SEARCH IN ROTATED SORTED ARRAY =============
// Time: O(log n) - modified binary search
// Space: O(1) - only pointers
// KEY INSIGHT: At least one half is always sorted
// Example: [4,5,6,7,0,1,2] - pivot at index 4
function searchRotated(nums, target) {
    let left = 0;
    let right = nums.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        
        if (nums[mid] === target) {
            return mid;
        }
        
        // CRITICAL: Determine which half is sorted
        // WHY? We can only do binary search on sorted portion
        if (nums[left] <= nums[mid]) {
            // Left half is sorted
            // Example: [4,5,6,7,0,1,2], mid=3 (value 7)
            // Left half [4,5,6,7] is sorted
            
            // Check if target is in sorted left half
            // WHY these conditions? Target must be:
            // 1. >= left boundary (nums[left])
            // 2. < mid value (already checked mid !== target)
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;  // Search left half
            } else {
                left = mid + 1;   // Search right half
            }
        } else {
            // Right half is sorted
            // Example: [6,7,0,1,2,4,5], mid=3 (value 1)
            // Right half [1,2,4,5] is sorted
            
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;   // Search right half
            } else {
                right = mid - 1;  // Search left half
            }
        }
    }
    
    return -1;
}

// ============= FIND PEAK ELEMENT =============
// Time: O(log n) - binary search variant
// Space: O(1) - only pointers
// KEY INSIGHT: Peak must exist - edges are -∞
// A peak is where nums[i] > nums[i-1] AND nums[i] > nums[i+1]
function findPeakElement(nums) {
    let left = 0;
    let right = nums.length - 1;
    
    // WHY left < right and not left <= right?
    // We need at least 2 elements to compare
    // When left === right, we've found the peak
    while (left < right) {
        const mid = Math.floor((left + right) / 2);
        
        // Compare mid with next element
        // WHY mid+1? We know it exists (left < right guarantees it)
        if (nums[mid] > nums[mid + 1]) {
            // We're on descending slope
            // Peak is on left side (could be mid itself)
            // WHY include mid? It might be the peak
            right = mid;
        } else {
            // We're on ascending slope (nums[mid] < nums[mid + 1])
            // Peak must be on right side
            // WHY mid + 1? We know mid is not peak (smaller than mid+1)
            left = mid + 1;
        }
    }
    
    // When left === right, we've converged to peak
    return left;
}

// ============= SEARCH INSERT POSITION =============
// Time: O(log n) - standard binary search
// Space: O(1) - only pointers
// Find index where target would be inserted to maintain order
// Example: [1,3,5,6], target=2 → return 1
function searchInsert(nums, target) {
    let left = 0;
    let right = nums.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        
        if (nums[mid] === target) {
            return mid;  // Exact match found
        }
        
        if (nums[mid] < target) {
            left = mid + 1;  // Target goes to right of mid
        } else {
            right = mid - 1;  // Target goes to left of mid
        }
    }
    
    // KEY INSIGHT: When loop ends, left is the insertion position
    // WHY? At termination:
    // - right = left - 1
    // - All elements before left are < target
    // - All elements from left onwards are > target
    // Therefore, left is where target should be inserted
    return left;
}
```

## Sorting Algorithms

### Comparison-Based Sorting

```javascript
// ============= BUBBLE SORT =============
// Time: O(n²) - WHY?
// Outer loop: n-1 iterations
// Inner loop: n-1, n-2, n-3, ... 1 iterations
// Total: (n-1) + (n-2) + ... + 1 = n(n-1)/2 = O(n²)
// Space: O(1) - sorting in place
// Stable: Yes - equal elements maintain relative order
function bubbleSort(arr) {
    const n = arr.length;
    
    // Each pass bubbles largest element to end
    for (let i = 0; i < n - 1; i++) {
        let swapped = false;  // Optimization flag
        
        // Compare adjacent elements
        // WHY n - i - 1?
        // After i passes, last i elements are sorted
        // No need to check them again
        for (let j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                // Swap using destructuring
                [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                swapped = true;
            }
        }
        
        // OPTIMIZATION: Early termination
        // If no swaps occurred, array is sorted
        // Best case becomes O(n) for already sorted
        if (!swapped) break;
    }
    
    return arr;
}

// ============= SELECTION SORT =============
// Time: O(n²) - WHY?
// Outer loop: n-1 iterations
// Inner loop: (n-1) + (n-2) + ... + 1 = n(n-1)/2 = O(n²)
// Space: O(1) - sorting in place
// Stable: No - swapping can change relative order
// KEY IDEA: Select minimum element and place at beginning
function selectionSort(arr) {
    const n = arr.length;
    
    // WHY i < n - 1 and not i < n?
    // Last element automatically in correct position
    // After n-1 iterations, array is sorted
    for (let i = 0; i < n - 1; i++) {
        let minIdx = i;  // Assume current position has minimum
        
        // Find actual minimum in remaining unsorted portion
        // WHY j = i + 1? Elements before i are already sorted
        for (let j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;  // Update minimum index
            }
        }
        
        // Swap minimum with current position
        // WHY check minIdx !== i?
        // Avoid unnecessary swap if element already in place
        if (minIdx !== i) {
            [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
        }
    }
    
    return arr;
}

// WHY NOT STABLE?
// Example: [4a, 2, 4b, 1, 3]
// After sorting: [1, 2, 3, 4b, 4a]
// 4a and 4b switched positions - stability lost

// ============= INSERTION SORT =============
// Time: O(n²) worst/average, O(n) best - WHY?
// Best case: Already sorted, only n-1 comparisons
// Worst case: Reverse sorted, 1+2+...+(n-1) = n(n-1)/2
// Space: O(1) - sorting in place
// Stable: Yes - only swap when strictly greater
// BEST FOR: Small arrays, nearly sorted data
function insertionSort(arr) {
    // Start from second element (first is trivially sorted)
    for (let i = 1; i < arr.length; i++) {
        const key = arr[i];  // Element to insert
        let j = i - 1;       // Start of sorted portion
        
        // Shift elements right until correct position found
        // WHY arr[j] > key and not arr[j] >= key?
        // Using > maintains stability (equal elements keep order)
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];  // Shift element right
            j--;                  // Check previous element
        }
        
        // Insert key at correct position
        // WHY j + 1? Loop ended when arr[j] <= key or j = -1
        // So key goes after arr[j], which is position j + 1
        arr[j + 1] = key;
    }
    
    return arr;
}

// WHY GOOD FOR NEARLY SORTED?
// Example: [2, 3, 4, 1, 5]
// Only element 1 needs significant movement
// Most elements require minimal/no shifting
// Results in O(n) time for nearly sorted data
```

### Efficient Sorting Algorithms

```javascript
// ============= MERGE SORT =============
// Time: O(n log n) - WHY?
// Divide: log n levels (halving each time)
// Conquer: n work per level (merging)
// Total: n * log n
// Space: O(n) - need auxiliary array for merging
// Stable: Yes - using <= in merge maintains order
// GUARANTEED O(n log n) - no worst case degradation
function mergeSort(arr) {
    // Base case: single element or empty array
    if (arr.length <= 1) return arr;
    
    // DIVIDE: Split array in half
    const mid = Math.floor(arr.length / 2);
    // slice creates new arrays - contributes to O(n) space
    const left = mergeSort(arr.slice(0, mid));
    const right = mergeSort(arr.slice(mid));
    
    // CONQUER: Merge sorted halves
    return merge(left, right);
}

// ============= MERGE HELPER =============
// Time: O(n) - single pass through both arrays
// Space: O(n) - result array
function merge(left, right) {
    const result = [];
    let i = 0, j = 0;  // Pointers for left and right arrays
    
    // Merge while both arrays have elements
    while (i < left.length && j < right.length) {
        // WHY <= and not <?
        // Using <= maintains stability
        // Equal elements from left array come first
        if (left[i] <= right[j]) {
            result.push(left[i++]);  // Take from left, advance pointer
        } else {
            result.push(right[j++]);  // Take from right, advance pointer
        }
    }
    
    // Append remaining elements (at most one array has elements)
    // WHY slice(i) and slice(j)?
    // One pointer reached end, other might have elements left
    return result.concat(left.slice(i)).concat(right.slice(j));
}

// RECURSION TREE ANALYSIS:
//        [8,3,5,4,7,6,1,2]
//       /                \
//   [8,3,5,4]          [7,6,1,2]
//    /     \            /     \
//  [8,3]   [5,4]     [7,6]   [1,2]
//  /  \    /  \      /  \    /  \
// [8] [3] [5] [4]  [7] [6] [1] [2]
// 
// Merging back up: O(n) work per level, log n levels

// ============= QUICK SORT =============
// Time: O(n log n) average, O(n²) worst - WHY?
// Best/Average: Balanced partitions → log n levels × n work
// Worst: Unbalanced (already sorted) → n levels × n work
// Space: O(log n) average, O(n) worst - recursion stack
// Stable: No - partitioning disrupts relative order
// IN-PLACE: Yes (unlike merge sort)
function quickSort(arr, low = 0, high = arr.length - 1) {
    // Base case: single element or invalid range
    if (low < high) {
        // PARTITION: Rearrange array and get pivot index
        // After partition: elements < pivot | pivot | elements > pivot
        const pi = partition(arr, low, high);
        
        // RECURSIVE CALLS: Sort left and right subarrays
        // WHY pi - 1 and pi + 1?
        // Pivot is already in correct position at index pi
        quickSort(arr, low, pi - 1);   // Sort left of pivot
        quickSort(arr, pi + 1, high);  // Sort right of pivot
    }
    return arr;
}

// ============= PARTITION (LOMUTO SCHEME) =============
// Rearranges array so pivot is in correct position
// Returns final position of pivot
function partition(arr, low, high) {
    // RANDOMIZED PIVOT - WHY?
    // Avoid O(n²) worst case on sorted arrays
    // Random pivot gives O(n log n) expected time
    const randomIdx = Math.floor(Math.random() * (high - low + 1)) + low;
    [arr[randomIdx], arr[high]] = [arr[high], arr[randomIdx]];
    
    const pivot = arr[high];  // Use last element as pivot
    let i = low - 1;  // Index of smaller element boundary
    
    // Traverse array, moving smaller elements to left
    for (let j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;  // Expand "smaller" region
            // Swap current element into "smaller" region
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
    }
    
    // Place pivot in correct position
    // WHY i + 1? 
    // i points to last element < pivot
    // i + 1 is first element >= pivot
    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
    return i + 1;  // Return pivot's final position
}

// PARTITION INVARIANT:
// arr[low...i] contains elements < pivot
// arr[i+1...j-1] contains elements >= pivot
// arr[j...high-1] contains unprocessed elements
// arr[high] is the pivot

// ============= HEAP SORT =============
// Time: O(n log n) - WHY?
// Build heap: O(n) - surprisingly not O(n log n)!
// Extract n elements: n × O(log n) = O(n log n)
// Space: O(1) - in-place sorting
// Stable: No - heap operations can change relative order
// CONSISTENT O(n log n) - no worst case degradation
function heapSort(arr) {
    const n = arr.length;
    
    // PHASE 1: Build max heap from array
    // WHY start at n/2 - 1?
    // Nodes from n/2 onwards are leaves (have no children)
    // Leaves are already valid heaps (single node)
    // We only need to heapify non-leaf nodes
    for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }
    
    // PHASE 2: Extract elements from heap one by one
    // Start from last element and work backwards
    for (let i = n - 1; i > 0; i--) {
        // Move current root (max) to end
        // This places largest element at correct position
        [arr[0], arr[i]] = [arr[i], arr[0]];
        
        // Heapify reduced heap (excluding sorted elements)
        // WHY pass i as size? 
        // Elements from i onwards are sorted, not part of heap
        heapify(arr, i, 0);
    }
    
    return arr;
}

// ============= HEAPIFY (SINK DOWN) =============
// Ensures subtree rooted at index i is a valid heap
// Time: O(log n) - maximum height of tree
function heapify(arr, n, i) {
    let largest = i;  // Initialize largest as root
    const left = 2 * i + 1;   // Left child index
    const right = 2 * i + 2;  // Right child index
    
    // HEAP PROPERTY: Parent >= Children (for max heap)
    
    // Check if left child exists and is larger than root
    if (left < n && arr[left] > arr[largest]) {
        largest = left;
    }
    
    // Check if right child exists and is larger than current largest
    if (right < n && arr[right] > arr[largest]) {
        largest = right;
    }
    
    // If largest is not root, swap and continue heapifying
    if (largest !== i) {
        // Swap root with largest child
        [arr[i], arr[largest]] = [arr[largest], arr[i]];
        
        // Recursively heapify affected subtree
        // WHY recursive? Swap might violate heap property below
        heapify(arr, n, largest);
    }
}

// WHY BUILD HEAP IS O(n) NOT O(n log n)?
// Mathematical proof: Sum of heights of all nodes
// Most nodes are near bottom with small heights
// Sum = n/2 × 0 + n/4 × 1 + n/8 × 2 + ... = O(n)
```

### Non-Comparison Sorting

```javascript
// ============= COUNTING SORT =============
// Time: O(n + k) - WHY?
// Count occurrences: O(n)
// Build cumulative: O(k)  
// Place elements: O(n)
// Total: O(n + k)
// Space: O(k) for count array + O(n) for output
// Stable: Yes - backward traversal maintains order
// LIMITATION: Only works for non-negative integers with known range
function countingSort(arr, maxVal = null) {
    if (arr.length === 0) return arr;
    
    // Find maximum value if not provided
    if (maxVal === null) {
        maxVal = Math.max(...arr);
    }
    
    // STEP 1: Count occurrences of each value
    // Index represents value, content represents count
    const count = new Array(maxVal + 1).fill(0);
    const output = new Array(arr.length);
    
    // Count each element
    // Example: [3,1,3,2] → count = [0,1,1,2] (0 zeros, 1 one, 1 two, 2 threes)
    for (const num of arr) {
        count[num]++;
    }
    
    // STEP 2: Transform to cumulative count
    // count[i] = number of elements <= i
    // This tells us ending position for each value
    for (let i = 1; i <= maxVal; i++) {
        count[i] += count[i - 1];
    }
    // Example: [0,1,1,2] → [0,1,2,4] (positions where each value group ends)
    
    // STEP 3: Place elements in sorted order
    // WHY traverse backward? To maintain stability
    // Elements with same value keep their relative order
    for (let i = arr.length - 1; i >= 0; i--) {
        // count[arr[i]] tells us where this value group ends
        // -1 because arrays are 0-indexed
        output[count[arr[i]] - 1] = arr[i];
        
        // Decrement count for next occurrence of same value
        // Next same value goes one position earlier
        count[arr[i]]--;
    }
    
    return output;
}

// WHY LINEAR TIME?
// No comparisons! We use array indices as implicit sorting
// Trade-off: Uses O(k) extra space where k is range
// Excellent when k = O(n), terrible when k >> n

// ============= RADIX SORT =============
// Time: O(d * (n + k)) - WHY?
// d passes (one per digit position)
// Each pass: O(n + k) counting sort (k=10 for decimal)
// Total: d × (n + 10) = O(d × n) for decimal numbers
// Space: O(n + k) for counting sort arrays
// Stable: Yes - uses stable counting sort
// WORKS FOR: Non-negative integers
function radixSort(arr) {
    if (arr.length === 0) return arr;
    
    // Find maximum to determine number of digits
    const max = Math.max(...arr);
    // Calculate digits: log10(123) = 2.09, floor + 1 = 3 digits
    const maxDigits = Math.floor(Math.log10(max)) + 1;
    
    // Sort by each digit position, starting from least significant
    // WHY start from ones place?
    // LSD (Least Significant Digit) radix sort
    // Stability ensures previously sorted digits remain sorted
    for (let digit = 0; digit < maxDigits; digit++) {
        arr = countingSortByDigit(arr, digit);
    }
    
    return arr;
}

// ============= COUNTING SORT BY DIGIT =============
// Modified counting sort that sorts by specific digit position
function countingSortByDigit(arr, digit) {
    const output = new Array(arr.length);
    const count = new Array(10).fill(0);  // 0-9 for decimal digits
    const digitPlace = Math.pow(10, digit);  // 1, 10, 100, ...
    
    // STEP 1: Count occurrences of each digit at position
    for (const num of arr) {
        // Extract digit at position
        // Example: num=123, digit=1 (tens)
        // 123 / 10 = 12.3, floor = 12, 12 % 10 = 2
        const digitValue = Math.floor(num / digitPlace) % 10;
        count[digitValue]++;
    }
    
    // STEP 2: Cumulative count for positions
    for (let i = 1; i < 10; i++) {
        count[i] += count[i - 1];
    }
    
    // STEP 3: Place elements maintaining stability
    // Backward traversal preserves order of equal digits
    for (let i = arr.length - 1; i >= 0; i--) {
        const digitValue = Math.floor(arr[i] / digitPlace) % 10;
        output[count[digitValue] - 1] = arr[i];
        count[digitValue]--;
    }
    
    return output;
}

// EXAMPLE WALKTHROUGH: [170, 45, 75, 90, 2, 802, 24, 66]
// Pass 1 (ones): [170, 90, 2, 802, 24, 45, 75, 66]
// Pass 2 (tens): [2, 802, 24, 45, 66, 170, 75, 90]
// Pass 3 (hundreds): [2, 24, 45, 66, 75, 90, 170, 802]

// ============= BUCKET SORT =============
// Time: O(n + k) average, O(n²) worst - WHY?
// Best: Uniform distribution, O(1) elements per bucket
// Worst: All elements in one bucket, degrades to O(n²)
// Space: O(n + k) for buckets
// Stable: Depends on sub-sorting algorithm
// BEST FOR: Uniformly distributed data over known range
function bucketSort(arr, bucketCount = 10) {
    if (arr.length === 0) return arr;
    
    // Find range of input
    const min = Math.min(...arr);
    const max = Math.max(...arr);
    const range = max - min;
    
    // Calculate bucket size
    // Each bucket covers bucketSize range of values
    const bucketSize = Math.ceil(range / bucketCount) || 1;
    
    // STEP 1: Create empty buckets
    const buckets = Array(bucketCount).fill(null).map(() => []);
    
    // STEP 2: Distribute elements into buckets
    // Elements are placed based on their value range
    for (const num of arr) {
        // Calculate which bucket this number belongs to
        // (num - min) normalizes to start from 0
        // Divide by bucketSize to get bucket index
        const bucketIdx = Math.floor((num - min) / bucketSize);
        
        // Handle edge case: max value would go to bucketCount index
        // Place it in last bucket instead
        buckets[Math.min(bucketIdx, bucketCount - 1)].push(num);
    }
    
    // STEP 3: Sort individual buckets and concatenate
    const result = [];
    for (const bucket of buckets) {
        // Use insertion sort for small buckets (efficient for small arrays)
        // Could use quicksort for larger buckets
        bucket.sort((a, b) => a - b);
        result.push(...bucket);  // Spread operator to concatenate
    }
    
    return result;
}

// WHY BUCKET SORT?
// When data is uniformly distributed:
// - Each bucket gets ~n/k elements
// - Sorting each: O((n/k)²) = O(n²/k²)
// - k buckets: k × O(n²/k²) = O(n²/k)
// - If k = n: O(n)
// Perfect for: Floating point numbers in known range
```

## Sorting Algorithm Comparison

| Algorithm | Time (Best) | Time (Avg) | Time (Worst) | Space | Stable | Notes |
|-----------|------------|------------|--------------|-------|--------|-------|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes | Simple, good for small data |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | No | Minimal swaps |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | Yes | Good for nearly sorted |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Consistent performance |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | No | Fastest average case |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No | In-place, consistent |
| Counting | O(n + k) | O(n + k) | O(n + k) | O(k) | Yes | Limited to integers |
| Radix | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | Yes | Good for integers |
| Bucket | O(n + k) | O(n + k) | O(n²) | O(n+k) | Yes | Good for uniform dist |

## AWS Applications

### 1. CloudWatch Logs Insights Query Optimization

```javascript
class LogQueryOptimizer {
    constructor() {
        this.indexes = new Map();
        this.sortedTimestamps = [];
    }
    
    // Build indexes for efficient querying
    buildIndexes(logs) {
        // Time-based index (sorted)
        this.sortedTimestamps = logs
            .map((log, idx) => ({ timestamp: log.timestamp, index: idx }))
            .sort((a, b) => a.timestamp - b.timestamp);
        
        // Field-based indexes
        for (const log of logs) {
            for (const [field, value] of Object.entries(log)) {
                if (!this.indexes.has(field)) {
                    this.indexes.set(field, new Map());
                }
                
                const fieldIndex = this.indexes.get(field);
                if (!fieldIndex.has(value)) {
                    fieldIndex.set(value, []);
                }
                fieldIndex.get(value).push(log);
            }
        }
    }
    
    // Binary search for time range
    queryTimeRange(startTime, endTime) {
        const startIdx = this.binarySearchTime(startTime, true);
        const endIdx = this.binarySearchTime(endTime, false);
        
        const results = [];
        for (let i = startIdx; i <= endIdx; i++) {
            results.push(this.sortedTimestamps[i].index);
        }
        
        return results;
    }
    
    binarySearchTime(targetTime, findFirst) {
        let left = 0;
        let right = this.sortedTimestamps.length - 1;
        let result = findFirst ? this.sortedTimestamps.length : -1;
        
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            const midTime = this.sortedTimestamps[mid].timestamp;
            
            if (findFirst) {
                if (midTime >= targetTime) {
                    result = mid;
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            } else {
                if (midTime <= targetTime) {
                    result = mid;
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }
        
        return result;
    }
    
    // Query with sorting
    query(filters, sortField, limit = 100) {
        let results = [];
        
        // Apply filters using indexes
        for (const [field, value] of Object.entries(filters)) {
            if (this.indexes.has(field)) {
                const fieldIndex = this.indexes.get(field);
                if (fieldIndex.has(value)) {
                    const matches = fieldIndex.get(value);
                    if (results.length === 0) {
                        results = matches;
                    } else {
                        // Intersection
                        results = results.filter(r => matches.includes(r));
                    }
                }
            }
        }
        
        // Sort results
        if (sortField) {
            results.sort((a, b) => {
                const aVal = a[sortField];
                const bVal = b[sortField];
                return aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
            });
        }
        
        // Apply limit
        return results.slice(0, limit);
    }
}

// Usage
const optimizer = new LogQueryOptimizer();
const logs = [
    { timestamp: 1609459200, level: 'ERROR', service: 'api', message: 'Connection failed' },
    { timestamp: 1609459201, level: 'INFO', service: 'api', message: 'Request received' },
    // ... more logs
];

optimizer.buildIndexes(logs);
const errorLogs = optimizer.query({ level: 'ERROR' }, 'timestamp', 10);
```

### 2. DynamoDB Query Optimization

```javascript
class DynamoQueryOptimizer {
    constructor() {
        this.partitions = new Map();
    }
    
    // Organize data by partition for efficient querying
    indexData(items) {
        for (const item of items) {
            const pk = item.pk;
            if (!this.partitions.has(pk)) {
                this.partitions.set(pk, []);
            }
            this.partitions.get(pk).push(item);
        }
        
        // Sort items within each partition by sort key
        for (const [pk, items] of this.partitions) {
            items.sort((a, b) => {
                if (a.sk < b.sk) return -1;
                if (a.sk > b.sk) return 1;
                return 0;
            });
        }
    }
    
    // Binary search within partition
    queryPartition(pk, skStart, skEnd) {
        if (!this.partitions.has(pk)) {
            return [];
        }
        
        const items = this.partitions.get(pk);
        
        // Find start index
        let startIdx = 0;
        if (skStart) {
            startIdx = this.binarySearchSK(items, skStart, true);
        }
        
        // Find end index
        let endIdx = items.length - 1;
        if (skEnd) {
            endIdx = this.binarySearchSK(items, skEnd, false);
        }
        
        return items.slice(startIdx, endIdx + 1);
    }
    
    binarySearchSK(items, target, findFirst) {
        let left = 0;
        let right = items.length - 1;
        let result = findFirst ? items.length : -1;
        
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            
            if (findFirst) {
                if (items[mid].sk >= target) {
                    result = mid;
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            } else {
                if (items[mid].sk <= target) {
                    result = mid;
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }
        
        return result;
    }
    
    // Parallel query across partitions
    parallelScan(filterFn, limit = 1000) {
        const results = [];
        const partitionKeys = Array.from(this.partitions.keys());
        
        // Simulate parallel processing
        const promises = partitionKeys.map(pk => {
            return new Promise(resolve => {
                const items = this.partitions.get(pk);
                const filtered = items.filter(filterFn);
                resolve(filtered);
            });
        });
        
        return Promise.all(promises).then(partitionResults => {
            // Merge results from all partitions
            const merged = partitionResults.flat();
            
            // Sort merged results if needed
            merged.sort((a, b) => a.timestamp - b.timestamp);
            
            return merged.slice(0, limit);
        });
    }
}
```

### 3. Athena Query Optimization

```javascript
class AthenaQueryOptimizer {
    constructor() {
        this.partitionPruning = true;
        this.columnPruning = true;
    }
    
    // Optimize query plan
    optimizeQuery(query) {
        let optimized = query;
        
        // Partition pruning
        if (this.partitionPruning) {
            optimized = this.addPartitionFilters(optimized);
        }
        
        // Column pruning
        if (this.columnPruning) {
            optimized = this.selectOnlyNeededColumns(optimized);
        }
        
        // Sort optimization
        optimized = this.optimizeSorting(optimized);
        
        return optimized;
    }
    
    // Add partition filters to reduce data scanned
    addPartitionFilters(query) {
        // Parse WHERE clause to identify partition columns
        const partitionColumns = ['year', 'month', 'day'];
        let optimized = query;
        
        for (const col of partitionColumns) {
            if (!query.includes(`WHERE ${col}`) && !query.includes(`AND ${col}`)) {
                // Suggest adding partition filter
                console.log(`Consider filtering on partition column: ${col}`);
            }
        }
        
        return optimized;
    }
    
    // Select only needed columns
    selectOnlyNeededColumns(query) {
        if (query.includes('SELECT *')) {
            console.log('Avoid SELECT *, specify needed columns instead');
        }
        return query;
    }
    
    // Optimize sorting operations
    optimizeSorting(query) {
        // Check for unnecessary sorting
        if (query.includes('ORDER BY') && !query.includes('LIMIT')) {
            console.log('Consider adding LIMIT when using ORDER BY');
        }
        
        // Check for sorting on non-indexed columns
        if (query.includes('ORDER BY')) {
            console.log('Ensure ORDER BY columns are indexed or partitioned');
        }
        
        return query;
    }
    
    // Estimate query cost
    estimateCost(dataSize, query) {
        let scanSize = dataSize;
        
        // Reduce by partition pruning
        if (query.includes('WHERE year') || query.includes('WHERE month')) {
            scanSize *= 0.1; // Assume 10% of data per partition
        }
        
        // Column pruning savings
        if (!query.includes('SELECT *')) {
            scanSize *= 0.3; // Assume 30% of columns selected
        }
        
        // Cost calculation (simplified)
        const costPerTB = 5; // $5 per TB scanned
        const costInDollars = (scanSize / 1e12) * costPerTB;
        
        return {
            scanSize,
            estimatedCost: costInDollars,
            optimizationSuggestions: this.getOptimizationSuggestions(query)
        };
    }
    
    getOptimizationSuggestions(query) {
        const suggestions = [];
        
        if (!query.includes('WHERE')) {
            suggestions.push('Add WHERE clause to filter data');
        }
        
        if (query.includes('SELECT *')) {
            suggestions.push('Select only needed columns');
        }
        
        if (!query.includes('LIMIT')) {
            suggestions.push('Add LIMIT to reduce result set');
        }
        
        return suggestions;
    }
}
```

## Practice Problems

### Problem 1: Search in 2D Matrix
```javascript
/**
 * Search in row-wise and column-wise sorted matrix
 * Time: O(m + n), Space: O(1)
 */
function searchMatrix(matrix, target) {
    if (!matrix.length || !matrix[0].length) return false;
    
    let row = 0;
    let col = matrix[0].length - 1;
    
    while (row < matrix.length && col >= 0) {
        if (matrix[row][col] === target) {
            return true;
        }
        
        if (matrix[row][col] > target) {
            col--;
        } else {
            row++;
        }
    }
    
    return false;
}
```

### Problem 2: Kth Largest Element
```javascript
/**
 * Find kth largest using quickselect
 * Time: O(n) average, O(n²) worst
 * Space: O(1)
 */
function findKthLargest(nums, k) {
    return quickSelect(nums, 0, nums.length - 1, nums.length - k);
}

function quickSelect(nums, left, right, k) {
    if (left === right) return nums[left];
    
    const pivotIndex = partition(nums, left, right);
    
    if (k === pivotIndex) {
        return nums[k];
    } else if (k < pivotIndex) {
        return quickSelect(nums, left, pivotIndex - 1, k);
    } else {
        return quickSelect(nums, pivotIndex + 1, right, k);
    }
}
```

### Problem 3: Merge K Sorted Arrays
```javascript
/**
 * Merge k sorted arrays using min heap
 * Time: O(n log k), Space: O(k)
 * where n is total elements
 */
function mergeKArrays(arrays) {
    const minHeap = new PriorityQueue((a, b) => a.val < b.val);
    const result = [];
    
    // Initialize heap with first element from each array
    for (let i = 0; i < arrays.length; i++) {
        if (arrays[i].length > 0) {
            minHeap.enqueue({
                val: arrays[i][0],
                arrayIdx: i,
                elementIdx: 0
            }, arrays[i][0]);
        }
    }
    
    while (!minHeap.isEmpty()) {
        const { val, arrayIdx, elementIdx } = minHeap.dequeue();
        result.push(val);
        
        // Add next element from same array
        if (elementIdx + 1 < arrays[arrayIdx].length) {
            const nextVal = arrays[arrayIdx][elementIdx + 1];
            minHeap.enqueue({
                val: nextVal,
                arrayIdx,
                elementIdx: elementIdx + 1
            }, nextVal);
        }
    }
    
    return result;
}
```

## Interview Tips

### Common Questions
1. **"Why is quicksort preferred over mergesort?"**
   - Better cache performance (in-place)
   - Lower space complexity O(log n) vs O(n)
   - Slightly better constants in practice

2. **"When would you use counting sort?"**
   - Small integer range
   - Need stable sort
   - Linear time is critical
   - Memory is available for count array

3. **"How do you handle sorting large datasets?"**
   - External sorting (merge sort with disk)
   - Distributed sorting (MapReduce)
   - Sampling for approximate sorting

### Red Flags to Avoid
- ❌ Using wrong algorithm for data characteristics
- ❌ Not considering stability requirements
- ❌ Ignoring space complexity constraints
- ❌ Forgetting about worst-case scenarios

### AWS-Specific Follow-ups
- "How does Athena optimize sort operations?"
- "Design a distributed sorting system"
- "How would you sort TB of data in S3?"
- "Optimize DynamoDB query sorting"

## Key Takeaways
1. **Binary search** requires sorted data but provides O(log n) search
2. **Quicksort** is usually fastest but has O(n²) worst case
3. **Merge sort** guarantees O(n log n) and is stable
4. **Non-comparison sorts** can achieve O(n) for specific data
5. **AWS services** optimize queries through indexing and partitioning

---

*Previous: [← Queues and Stacks](./04-queues-and-stacks.md)*
*Next: [Divide and Conquer →](./06-divide-and-conquer.md)*
