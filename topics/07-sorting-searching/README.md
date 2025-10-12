# 🔍 Sorting & Searching

## Overview
Sorting and searching are fundamental algorithmic techniques. Sorting arranges data in order while searching finds specific elements. Together, they form the backbone of efficient data processing.

## Sorting Algorithms

### Comparison-Based Sorting
| Algorithm | Best | Average | Worst | Space | Stable | Notes |
|-----------|------|---------|-------|-------|---------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Simple, educational |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No | Minimal swaps |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Good for small/nearly sorted |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Divide & conquer |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No* | Usually fastest |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | In-place |

### Non-Comparison Sorting
| Algorithm | Time | Space | Use Case |
|-----------|------|-------|----------|
| Counting Sort | O(n + k) | O(k) | Small range integers |
| Radix Sort | O(d × (n + k)) | O(n + k) | Fixed-width integers |
| Bucket Sort | O(n + k) | O(n + k) | Uniform distribution |

## Searching Algorithms

### Linear Search
```javascript
function linearSearch(arr, target) {
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] === target) return i;
    }
    return -1;
}
// Time: O(n), Space: O(1)
```

### Binary Search
```javascript
function binarySearch(arr, target) {
    let left = 0, right = arr.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) return mid;
        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return -1;
}
// Time: O(log n), Space: O(1)
```

## Problems in This Section

### Basic
- [Insertion Sort](problems/insertion-sort.md) - Step-by-step sorting
- [Quicksort Partition](problems/quicksort-partition.md) - Partition algorithm

### Intermediate
- [Search 2D Matrix](problems/search-2d-matrix.md) - Binary search variant
- [Kth Largest Element](problems/kth-largest-element.md) - Quick select
- [Merge K Sorted Arrays](problems/merge-k-sorted-arrays.md) - K-way merge

### Advanced
- [Median of Two Sorted Arrays](problems/find-median-sorted-arrays.md) - Binary search application

## Implementation Examples

### Quick Sort
```javascript
function quickSort(arr, low = 0, high = arr.length - 1) {
    if (low < high) {
        const pivot = partition(arr, low, high);
        quickSort(arr, low, pivot - 1);
        quickSort(arr, pivot + 1, high);
    }
    return arr;
}

function partition(arr, low, high) {
    const pivot = arr[high];
    let i = low - 1;
    
    for (let j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
    }
    
    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
    return i + 1;
}
```

### Merge Sort
```javascript
function mergeSort(arr) {
    if (arr.length <= 1) return arr;
    
    const mid = Math.floor(arr.length / 2);
    const left = mergeSort(arr.slice(0, mid));
    const right = mergeSort(arr.slice(mid));
    
    return merge(left, right);
}

function merge(left, right) {
    const result = [];
    let i = 0, j = 0;
    
    while (i < left.length && j < right.length) {
        if (left[i] <= right[j]) {
            result.push(left[i++]);
        } else {
            result.push(right[j++]);
        }
    }
    
    return result.concat(left.slice(i)).concat(right.slice(j));
}
```

## Binary Search Variants

### Find First/Last Occurrence
```javascript
function findFirst(arr, target) {
    let left = 0, right = arr.length - 1;
    let result = -1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) {
            result = mid;
            right = mid - 1; // Continue searching left
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}
```

### Search in Rotated Array
```javascript
function searchRotated(arr, target) {
    let left = 0, right = arr.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) return mid;
        
        // Left half is sorted
        if (arr[left] <= arr[mid]) {
            if (target >= arr[left] && target < arr[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        // Right half is sorted
        else {
            if (target > arr[mid] && target <= arr[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    
    return -1;
}
```

## Advanced Techniques

### Quick Select (Find kth element)
```javascript
function quickSelect(arr, k, left = 0, right = arr.length - 1) {
    if (left === right) return arr[left];
    
    const pivotIndex = partition(arr, left, right);
    
    if (k === pivotIndex) {
        return arr[k];
    } else if (k < pivotIndex) {
        return quickSelect(arr, k, left, pivotIndex - 1);
    } else {
        return quickSelect(arr, k, pivotIndex + 1, right);
    }
}
// Average: O(n), Worst: O(n²)
```

### Three-Way Partition (Dutch Flag)
```javascript
function threeWayPartition(arr, pivot) {
    let low = 0, mid = 0, high = arr.length - 1;
    
    while (mid <= high) {
        if (arr[mid] < pivot) {
            [arr[low], arr[mid]] = [arr[mid], arr[low]];
            low++;
            mid++;
        } else if (arr[mid] > pivot) {
            [arr[mid], arr[high]] = [arr[high], arr[mid]];
            high--;
        } else {
            mid++;
        }
    }
    
    return arr;
}
```

## Tips for Success

### Sorting
1. **Know complexities** - Time and space for each algorithm
2. **Stability matters** - Preserve relative order of equal elements
3. **Nearly sorted data** - Insertion sort can be optimal
4. **Choose wisely** - Quick sort usually, merge sort for stability
5. **In-place vs extra space** - Trade-off decision

### Searching
1. **Sorted requirement** - Binary search needs sorted data
2. **Handle duplicates** - Find first/last occurrence
3. **Boundary conditions** - Empty array, single element
4. **Overflow prevention** - Use `left + (right - left) / 2`
5. **Variant recognition** - Rotated, 2D matrix, etc.

## Interview Patterns

- **Two pointers** - Often combined with sorting
- **Binary search on answer** - Min/max problems
- **Partial sorting** - Heap for k elements
- **Custom comparators** - Sort by specific criteria
- **Stability requirements** - When order matters

## When to Use

### Use Sorting When:
✅ Need ordered data
✅ Finding duplicates
✅ Two-pointer techniques
✅ Binary search preparation
✅ Interval problems

### Use Binary Search When:
✅ Sorted data available
✅ Looking for specific value
✅ Optimization problems
✅ Finding boundaries
✅ O(log n) required
