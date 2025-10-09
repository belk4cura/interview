# Divide and Conquer

## Overview
Divide and conquer is a powerful algorithmic paradigm that breaks problems into smaller subproblems, solves them independently, and combines results. It's the foundation for many efficient algorithms and distributed computing patterns.

### Key Concepts
- **Divide**: Break problem into smaller subproblems
- **Conquer**: Solve subproblems recursively
- **Combine**: Merge subproblem solutions
- **AWS Applications**: MapReduce on EMR, distributed processing, parallel computing

## Core Principles

### Divide and Conquer Structure
```javascript
// ============= DIVIDE AND CONQUER TEMPLATE =============
// Time Complexity Analysis using Master Theorem:
// T(n) = a*T(n/b) + f(n)
// Where:
// - a = number of subproblems
// - n/b = size of each subproblem
// - f(n) = time to divide and combine
function divideAndConquer(problem) {
    // BASE CASE: Problem small enough to solve directly
    // WHY? Recursion needs stopping condition
    // Usually when n = 0, 1, or small constant
    if (problem.isSmallEnough()) {
        return problem.solveDirect();  // O(1) or O(k) for small k
    }
    
    // DIVIDE: Break into smaller subproblems
    // Usually O(1) or O(log n) for finding split point
    const subproblems = problem.divide();
    
    // CONQUER: Solve subproblems recursively
    // Each recursive call adds to call stack
    // Maximum depth = log(n) for balanced division
    const solutions = subproblems.map(subproblem => 
        divideAndConquer(subproblem)
    );
    
    // COMBINE: Merge subproblem solutions
    // Complexity varies: O(1) for binary search, O(n) for merge sort
    return problem.combine(solutions);
}

// WHEN TO USE DIVIDE AND CONQUER:
// 1. Problem can be broken into similar subproblems
// 2. Subproblems are independent (can solve separately)
// 3. Combining solutions is efficient
// 4. Recursion tree is balanced (avoids O(n²))
```

## Classic Algorithms

### Merge Sort (Review)
```javascript
// ============= MERGE SORT - CLASSIC DIVIDE & CONQUER =============
// Time: O(n log n) - WHY?
// Recurrence: T(n) = 2T(n/2) + O(n)
// - Divide: O(1) to find middle
// - Conquer: 2 subproblems of size n/2
// - Combine: O(n) to merge
// Master Theorem: a=2, b=2, f(n)=n → O(n log n)
// Space: O(n) for merge arrays + O(log n) call stack
function mergeSort(arr) {
    // Base case: arrays of size 0 or 1 are already sorted
    if (arr.length <= 1) return arr;
    
    // DIVIDE: Split array in half - O(1)
    const mid = Math.floor(arr.length / 2);
    const left = arr.slice(0, mid);   // First half
    const right = arr.slice(mid);     // Second half
    
    // CONQUER: Recursively sort both halves
    // Each recursive call adds to call stack (log n depth)
    const sortedLeft = mergeSort(left);
    const sortedRight = mergeSort(right);
    
    // COMBINE: Merge sorted halves - O(n)
    return merge(sortedLeft, sortedRight);
}

// ============= MERGE HELPER =============
// Time: O(n) where n = left.length + right.length
// Space: O(n) for result array
function merge(left, right) {
    const result = [];
    let i = 0, j = 0;  // Pointers for left and right arrays
    
    // Compare elements from both arrays
    // Take smaller element each time
    while (i < left.length && j < right.length) {
        // <= ensures stability (equal elements maintain order)
        if (left[i] <= right[j]) {
            result.push(left[i++]);
        } else {
            result.push(right[j++]);
        }
    }
    
    // Append remaining elements (one array is exhausted)
    // Only one of these will execute
    return result.concat(left.slice(i)).concat(right.slice(j));
}

// WHY IS MERGE SORT EFFICIENT?
// 1. Predictable O(n log n) - no worst case like quicksort
// 2. Stable - maintains relative order of equal elements  
// 3. Parallelizable - independent subproblems
// 4. External sorting - good for data that doesn't fit in memory
```

### Quick Sort (Review)
```javascript
// ============= QUICKSORT - DIVIDE & CONQUER WITH PIVOT =============
// Time: O(n log n) average, O(n²) worst - WHY?
// Recurrence: T(n) = T(k) + T(n-k-1) + O(n)
// Best case (balanced): k ≈ n/2 → T(n) = 2T(n/2) + O(n) = O(n log n)
// Worst case (unbalanced): k = 0 → T(n) = T(n-1) + O(n) = O(n²)
// Space: O(log n) average for recursion stack
function quickSort(arr, low = 0, high = arr.length - 1) {
    // Base case: single element or invalid range
    if (low < high) {
        // DIVIDE: Partition array around pivot
        // All elements < pivot go left, > pivot go right
        // O(n) time to partition
        const pi = partition(arr, low, high);
        
        // CONQUER: Recursively sort both halves
        // Pivot is already in correct position
        quickSort(arr, low, pi - 1);   // Sort left of pivot
        quickSort(arr, pi + 1, high);  // Sort right of pivot
    }
    return arr;
}

// ============= PARTITION SUBROUTINE =============
// Lomuto partition scheme - simpler but more swaps
// Time: O(n) - single pass through array
function partition(arr, low, high) {
    const pivot = arr[high];  // Choose last element as pivot
    let i = low - 1;  // Index of smaller element boundary
    
    // Traverse array, moving smaller elements left
    for (let j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;  // Expand smaller region
            [arr[i], arr[j]] = [arr[j], arr[i]];  // Swap
        }
    }
    
    // Place pivot in correct position
    // All elements left of i+1 are < pivot
    // All elements right of i+1 are >= pivot
    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
    return i + 1;  // Return pivot's final position
}

// WHY IS QUICKSORT PREFERRED DESPITE O(n²) WORST CASE?
// 1. In-place: O(1) extra space (excluding recursion)
// 2. Cache efficient: Good locality of reference
// 3. Average case O(n log n) is common in practice
// 4. Random pivot selection avoids worst case
```

### Binary Search (Divide and Conquer View)
```javascript
// ============= BINARY SEARCH - SIMPLEST DIVIDE & CONQUER =============
// Time: O(log n) - WHY?
// Recurrence: T(n) = T(n/2) + O(1)
// We solve ONE subproblem of half size + constant work
// By Master Theorem: a=1, b=2, f(n)=1 → O(log n)
// Space: O(log n) for recursion, O(1) iterative
function binarySearch(arr, target, left = 0, right = arr.length - 1) {
    // BASE CASE: Search space exhausted
    if (left > right) return -1;
    
    // DIVIDE: Find middle point - O(1)
    const mid = Math.floor((left + right) / 2);
    
    // CHECK: Is this our target? - O(1)
    if (arr[mid] === target) return mid;
    
    // CONQUER: Search ONE half (key difference from merge sort)
    // We can eliminate half because array is SORTED
    if (arr[mid] > target) {
        // Target must be in left half
        return binarySearch(arr, target, left, mid - 1);
    } else {
        // Target must be in right half
        return binarySearch(arr, target, mid + 1, right);
    }
}

// WHY IS BINARY SEARCH SO EFFICIENT?
// 1. We solve only ONE subproblem, not both
// 2. Combine step is trivial - just return result
// 3. Each level eliminates half the search space
// 4. After k steps, we've checked n/2^k elements
// 5. We're done when n/2^k = 1, so k = log n
```

## Advanced Divide and Conquer

### Maximum Subarray (Kadane's Alternative)
```javascript
// ============= MAX SUBARRAY - DIVIDE & CONQUER =============
// Time: O(n log n) - WHY?
// Recurrence: T(n) = 2T(n/2) + O(n)
// Two subproblems of half size + linear combine
// By Master Theorem: O(n log n)
// Space: O(log n) for recursion stack
// NOTE: Kadane's algorithm is O(n), but this shows D&C approach
function maxSubarrayDivideConquer(arr, left = 0, right = arr.length - 1) {
    // BASE CASE: Single element
    if (left === right) {
        return arr[left];
    }
    
    // DIVIDE: Split array at midpoint
    const mid = Math.floor((left + right) / 2);
    
    // CONQUER: Find max in each half
    // Max subarray could be entirely in left half
    const leftMax = maxSubarrayDivideConquer(arr, left, mid);
    // Or entirely in right half
    const rightMax = maxSubarrayDivideConquer(arr, mid + 1, right);
    
    // COMBINE: But it could also cross the middle!
    // This is the KEY INSIGHT - we need to check crossing case
    const crossMax = maxCrossingSum(arr, left, mid, right);
    
    // Return maximum of all three possibilities
    return Math.max(leftMax, rightMax, crossMax);
}

// ============= FIND MAX CROSSING SUBARRAY =============
// Time: O(n) - linear scan from middle outward
// KEY INSIGHT: A crossing subarray MUST include mid and mid+1
function maxCrossingSum(arr, left, mid, right) {
    // Find max sum extending left from mid
    // WHY start at mid? Crossing subarray must include it
    let leftSum = -Infinity;
    let sum = 0;
    for (let i = mid; i >= left; i--) {
        sum += arr[i];
        leftSum = Math.max(leftSum, sum);
    }
    
    // Find max sum extending right from mid+1
    // WHY start at mid+1? Must be contiguous with left part
    let rightSum = -Infinity;
    sum = 0;
    for (let i = mid + 1; i <= right; i++) {
        sum += arr[i];
        rightSum = Math.max(rightSum, sum);
    }
    
    // Combine left and right extensions
    return leftSum + rightSum;
}

// EXAMPLE: [-2, 1, -3, 4, -1, 2, 1, -5, 4]
// Divide at index 4:
// Left: [-2, 1, -3, 4, -1] → max = 4
// Right: [2, 1, -5, 4] → max = 4
// Cross: [...4, -1] + [2, 1...] = 6
// Result: max(4, 4, 6) = 6
```

### Closest Pair of Points
```javascript
// ============= CLOSEST PAIR - GEOMETRIC DIVIDE & CONQUER =============
// Time: O(n log n) - WHY?
// Initial sort: O(n log n)
// Recurrence: T(n) = 2T(n/2) + O(n) for strip check = O(n log n)
// Space: O(n) for strip array
function closestPair(points) {
    // Pre-sort by x-coordinate (one-time cost)
    // WHY? Enables O(1) divide step in recursion
    points.sort((a, b) => a.x - b.x);
    
    return closestPairRecursive(points, 0, points.length - 1);
}

// ============= RECURSIVE CLOSEST PAIR =============
function closestPairRecursive(points, left, right) {
    // BASE CASE: Small arrays use brute force
    // WHY 3? For ≤3 points, brute force is efficient
    // Also prevents infinite recursion
    if (right - left <= 3) {
        return bruteForceClosest(points, left, right);
    }
    
    // DIVIDE: Split points by vertical line through middle
    const mid = Math.floor((left + right) / 2);
    const midPoint = points[mid];  // Point at dividing line
    
    // CONQUER: Find closest pair in each half
    // Left half: points[left...mid]
    const leftMin = closestPairRecursive(points, left, mid);
    // Right half: points[mid+1...right]
    const rightMin = closestPairRecursive(points, mid + 1, right);
    
    // Current minimum from both halves
    let minDist = Math.min(leftMin, rightMin);
    
    // COMBINE: Check if closest pair crosses the middle line
    // KEY INSIGHT: Only need to check points within minDist of dividing line
    // WHY? Points farther apart can't be closer than current minimum
    const strip = [];
    for (let i = left; i <= right; i++) {
        // Include points within minDist of vertical dividing line
        if (Math.abs(points[i].x - midPoint.x) < minDist) {
            strip.push(points[i]);
        }
    }
    
    // Find closest points in strip (might cross dividing line)
    const stripMin = closestInStrip(strip, minDist);
    
    // Return overall minimum
    return Math.min(minDist, stripMin);
}

// ============= BRUTE FORCE FOR BASE CASE =============
// Time: O(n²) but n ≤ 3, so effectively O(1)
function bruteForceClosest(points, left, right) {
    let minDist = Infinity;
    
    // Check all pairs
    for (let i = left; i < right; i++) {
        for (let j = i + 1; j <= right; j++) {
            const dist = distance(points[i], points[j]);
            minDist = Math.min(minDist, dist);
        }
    }
    
    return minDist;
}

// ============= FIND CLOSEST IN STRIP =============
// Time: O(n) - despite nested loops! WHY?
// Geometric property: For any point, at most 7 other points
// can be within minDist in the strip
function closestInStrip(strip, minDist) {
    // Sort by y-coordinate for efficient checking
    strip.sort((a, b) => a.y - b.y);
    
    let min = minDist;
    
    // Check each point against nearby points
    for (let i = 0; i < strip.length; i++) {
        // OPTIMIZATION: Only check points with y-distance < min
        // WHY? If y-distance ≥ min, total distance must be ≥ min
        for (let j = i + 1; j < strip.length && 
             (strip[j].y - strip[i].y) < min; j++) {
            const dist = distance(strip[i], strip[j]);
            min = Math.min(min, dist);
        }
        // Inner loop runs at most 7 times (geometric proof)
    }
    
    return min;
}

// ============= EUCLIDEAN DISTANCE =============
function distance(p1, p2) {
    // Standard 2D distance formula
    return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
}

// GEOMETRIC INSIGHT:
// In the strip of width 2δ (δ on each side of dividing line),
// points are constrained. If we divide strip into δ×δ squares,
// each square can contain at most 1 point (else distance < δ).
// A point can only be close to points in adjacent squares,
// limiting comparisons to constant number.
```

### Matrix Multiplication (Strassen's Algorithm)
```javascript
/**
 * Strassen's matrix multiplication
 * Time: O(n^2.807), Space: O(n²)
 * Better than naive O(n³) for large matrices
 */
function strassenMultiply(A, B) {
    const n = A.length;
    
    // Base case: 1x1 matrix
    if (n === 1) {
        return [[A[0][0] * B[0][0]]];
    }
    
    // Ensure matrix size is power of 2
    if (n % 2 !== 0) {
        return standardMultiply(A, B); // Fallback
    }
    
    // Divide matrices into quadrants
    const mid = n / 2;
    const a11 = getQuadrant(A, 0, 0, mid);
    const a12 = getQuadrant(A, 0, mid, mid);
    const a21 = getQuadrant(A, mid, 0, mid);
    const a22 = getQuadrant(A, mid, mid, mid);
    
    const b11 = getQuadrant(B, 0, 0, mid);
    const b12 = getQuadrant(B, 0, mid, mid);
    const b21 = getQuadrant(B, mid, 0, mid);
    const b22 = getQuadrant(B, mid, mid, mid);
    
    // Calculate Strassen's 7 products
    const m1 = strassenMultiply(
        addMatrix(a11, a22),
        addMatrix(b11, b22)
    );
    const m2 = strassenMultiply(
        addMatrix(a21, a22),
        b11
    );
    const m3 = strassenMultiply(
        a11,
        subtractMatrix(b12, b22)
    );
    const m4 = strassenMultiply(
        a22,
        subtractMatrix(b21, b11)
    );
    const m5 = strassenMultiply(
        addMatrix(a11, a12),
        b22
    );
    const m6 = strassenMultiply(
        subtractMatrix(a21, a11),
        addMatrix(b11, b12)
    );
    const m7 = strassenMultiply(
        subtractMatrix(a12, a22),
        addMatrix(b21, b22)
    );
    
    // Combine results
    const c11 = addMatrix(
        subtractMatrix(addMatrix(m1, m4), m5),
        m7
    );
    const c12 = addMatrix(m3, m5);
    const c21 = addMatrix(m2, m4);
    const c22 = addMatrix(
        subtractMatrix(addMatrix(m1, m3), m2),
        m6
    );
    
    // Construct result matrix
    return combineQuadrants(c11, c12, c21, c22);
}

function getQuadrant(matrix, rowStart, colStart, size) {
    const quadrant = [];
    for (let i = 0; i < size; i++) {
        quadrant[i] = [];
        for (let j = 0; j < size; j++) {
            quadrant[i][j] = matrix[rowStart + i][colStart + j];
        }
    }
    return quadrant;
}

function addMatrix(A, B) {
    const n = A.length;
    const result = [];
    for (let i = 0; i < n; i++) {
        result[i] = [];
        for (let j = 0; j < n; j++) {
            result[i][j] = A[i][j] + B[i][j];
        }
    }
    return result;
}

function subtractMatrix(A, B) {
    const n = A.length;
    const result = [];
    for (let i = 0; i < n; i++) {
        result[i] = [];
        for (let j = 0; j < n; j++) {
            result[i][j] = A[i][j] - B[i][j];
        }
    }
    return result;
}

function combineQuadrants(c11, c12, c21, c22) {
    const n = c11.length * 2;
    const result = Array(n).fill(null).map(() => Array(n).fill(0));
    const mid = n / 2;
    
    // Copy quadrants to result
    for (let i = 0; i < mid; i++) {
        for (let j = 0; j < mid; j++) {
            result[i][j] = c11[i][j];
            result[i][j + mid] = c12[i][j];
            result[i + mid][j] = c21[i][j];
            result[i + mid][j + mid] = c22[i][j];
        }
    }
    
    return result;
}
```

## Parallel Divide and Conquer

### Parallel Merge Sort
```javascript
/**
 * Parallel merge sort simulation
 * Time: O(n log n), but faster wall-clock time
 */
class ParallelMergeSort {
    constructor(numWorkers = 4) {
        this.numWorkers = numWorkers;
    }
    
    async sort(arr) {
        if (arr.length <= 1) return arr;
        
        // Determine chunk size for each worker
        const chunkSize = Math.ceil(arr.length / this.numWorkers);
        const chunks = [];
        
        // Divide into chunks
        for (let i = 0; i < arr.length; i += chunkSize) {
            chunks.push(arr.slice(i, i + chunkSize));
        }
        
        // Parallel sort each chunk
        const sortPromises = chunks.map(chunk => 
            this.simulateWorkerSort(chunk)
        );
        
        const sortedChunks = await Promise.all(sortPromises);
        
        // Merge sorted chunks
        return this.mergeAllChunks(sortedChunks);
    }
    
    async simulateWorkerSort(chunk) {
        // Simulate async work
        return new Promise(resolve => {
            setTimeout(() => {
                resolve(chunk.sort((a, b) => a - b));
            }, Math.random() * 100);
        });
    }
    
    mergeAllChunks(chunks) {
        // Iteratively merge chunks
        while (chunks.length > 1) {
            const merged = [];
            
            for (let i = 0; i < chunks.length; i += 2) {
                if (i + 1 < chunks.length) {
                    merged.push(this.mergeTwoArrays(chunks[i], chunks[i + 1]));
                } else {
                    merged.push(chunks[i]);
                }
            }
            
            chunks = merged;
        }
        
        return chunks[0];
    }
    
    mergeTwoArrays(left, right) {
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
}

// Usage
const parallelSort = new ParallelMergeSort(4);
parallelSort.sort([64, 34, 25, 12, 22, 11, 90]).then(sorted => {
    console.log('Sorted:', sorted);
});
```

## AWS MapReduce Implementation

### EMR MapReduce Pattern
```javascript
/**
 * MapReduce implementation for distributed processing
 */
class MapReduceJob {
    constructor() {
        this.mappers = [];
        this.reducers = [];
    }
    
    // Map phase: process input in parallel
    map(data, mapFunction) {
        const promises = data.map(item => 
            new Promise(resolve => {
                // Simulate distributed processing
                setTimeout(() => {
                    const result = mapFunction(item);
                    resolve(result);
                }, Math.random() * 100);
            })
        );
        
        return Promise.all(promises);
    }
    
    // Shuffle and sort phase
    shuffleAndSort(mappedData) {
        const shuffled = new Map();
        
        // Group by key
        for (const pairs of mappedData) {
            for (const [key, value] of pairs) {
                if (!shuffled.has(key)) {
                    shuffled.set(key, []);
                }
                shuffled.get(key).push(value);
            }
        }
        
        // Sort keys
        const sortedKeys = Array.from(shuffled.keys()).sort();
        const sorted = new Map();
        
        for (const key of sortedKeys) {
            sorted.set(key, shuffled.get(key));
        }
        
        return sorted;
    }
    
    // Reduce phase: aggregate results
    async reduce(shuffledData, reduceFunction) {
        const promises = [];
        
        for (const [key, values] of shuffledData) {
            promises.push(
                new Promise(resolve => {
                    setTimeout(() => {
                        const result = reduceFunction(key, values);
                        resolve([key, result]);
                    }, Math.random() * 100);
                })
            );
        }
        
        const results = await Promise.all(promises);
        return new Map(results);
    }
    
    // Execute complete MapReduce job
    async execute(data, mapFunction, reduceFunction) {
        console.log('Starting Map phase...');
        const mapped = await this.map(data, mapFunction);
        
        console.log('Shuffling and sorting...');
        const shuffled = this.shuffleAndSort(mapped);
        
        console.log('Starting Reduce phase...');
        const reduced = await this.reduce(shuffled, reduceFunction);
        
        return reduced;
    }
}

// Example: Word count
async function wordCount(documents) {
    const job = new MapReduceJob();
    
    // Map function: emit word counts
    const mapFunction = (document) => {
        const words = document.toLowerCase().split(/\s+/);
        return words.map(word => [word, 1]);
    };
    
    // Reduce function: sum counts
    const reduceFunction = (word, counts) => {
        return counts.reduce((sum, count) => sum + count, 0);
    };
    
    const result = await job.execute(documents, mapFunction, reduceFunction);
    return result;
}

// Usage
const documents = [
    "Hello world from AWS",
    "Hello from EMR",
    "MapReduce on AWS EMR"
];

wordCount(documents).then(result => {
    console.log('Word counts:', result);
});
```

### Distributed Aggregation
```javascript
/**
 * Distributed aggregation using divide and conquer
 */
class DistributedAggregator {
    constructor(numNodes = 4) {
        this.numNodes = numNodes;
    }
    
    // Aggregate large dataset across nodes
    async aggregate(data, aggregateFunc, combineFunc) {
        // Divide data among nodes
        const chunkSize = Math.ceil(data.length / this.numNodes);
        const chunks = [];
        
        for (let i = 0; i < data.length; i += chunkSize) {
            chunks.push(data.slice(i, i + chunkSize));
        }
        
        // Process chunks in parallel (simulate nodes)
        const nodeResults = await Promise.all(
            chunks.map(chunk => this.processOnNode(chunk, aggregateFunc))
        );
        
        // Combine results from all nodes
        return this.combineResults(nodeResults, combineFunc);
    }
    
    async processOnNode(chunk, aggregateFunc) {
        // Simulate network delay
        await new Promise(resolve => 
            setTimeout(resolve, Math.random() * 100)
        );
        
        return chunk.reduce(aggregateFunc);
    }
    
    combineResults(results, combineFunc) {
        // Hierarchical combination (tree reduction)
        while (results.length > 1) {
            const combined = [];
            
            for (let i = 0; i < results.length; i += 2) {
                if (i + 1 < results.length) {
                    combined.push(combineFunc(results[i], results[i + 1]));
                } else {
                    combined.push(results[i]);
                }
            }
            
            results = combined;
        }
        
        return results[0];
    }
}

// Example: Distributed sum
const aggregator = new DistributedAggregator(4);

const data = Array.from({ length: 1000 }, (_, i) => i + 1);

aggregator.aggregate(
    data,
    (acc, val) => acc + val,  // Local aggregation
    (a, b) => a + b            // Combine results
).then(result => {
    console.log('Sum:', result); // 500500
});
```

## Practice Problems

### Problem 1: Count Inversions
```javascript
// ============= COUNT INVERSIONS - MODIFIED MERGE SORT =============
// Inversion: pair (i,j) where i < j but arr[i] > arr[j]
// Time: O(n log n) - same as merge sort
// Space: O(n) for temporary array
// KEY INSIGHT: Count inversions during merge step
function countInversions(arr) {
    // Temporary array for merging (avoids repeated allocation)
    const temp = new Array(arr.length);
    return mergeSortAndCount(arr, temp, 0, arr.length - 1);
}

// ============= MERGE SORT WITH INVERSION COUNTING =============
function mergeSortAndCount(arr, temp, left, right) {
    let invCount = 0;
    
    // Base case: single element has no inversions
    if (left < right) {
        const mid = Math.floor((left + right) / 2);
        
        // DIVIDE & CONQUER: Count inversions in each half
        // Inversions where both indices in left half
        invCount += mergeSortAndCount(arr, temp, left, mid);
        
        // Inversions where both indices in right half
        invCount += mergeSortAndCount(arr, temp, mid + 1, right);
        
        // COMBINE: Count inversions across halves
        // Inversions where left index in left half, right index in right half
        invCount += mergeAndCount(arr, temp, left, mid, right);
    }
    
    return invCount;
}

// ============= MERGE WITH INVERSION COUNTING =============
// Time: O(n) for merge + counting
function mergeAndCount(arr, temp, left, mid, right) {
    let i = left;    // Left subarray index
    let j = mid + 1; // Right subarray index
    let k = left;    // Merged array index
    let invCount = 0;
    
    // Merge while counting inversions
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) {
            // No inversion: left element ≤ right element
            temp[k++] = arr[i++];
        } else {
            // INVERSION FOUND! arr[i] > arr[j] but i < j
            temp[k++] = arr[j++];
            
            // KEY COUNTING LOGIC:
            // If arr[i] > arr[j], then all elements from
            // arr[i] to arr[mid] are also > arr[j]
            // WHY? Left subarray is sorted
            // So we have (mid - i + 1) inversions
            invCount += (mid - i + 1);
        }
    }
    
    // Copy remaining elements (no more inversions possible)
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    
    // Copy sorted elements back to original array
    for (i = left; i <= right; i++) {
        arr[i] = temp[i];
    }
    
    return invCount;
}

// EXAMPLE: [2, 4, 1, 3, 5]
// Left half: [2, 4] - 0 inversions
// Right half: [1, 3, 5] - 0 inversions
// Cross inversions during merge:
// - 2 > 1: inversions = (1-0+1) = 2 → (2,1) and (4,1)
// - 4 > 3: inversions = (1-1+1) = 1 → (4,3)
// Total: 0 + 0 + 3 = 3 inversions
```

### Problem 2: Median of Two Sorted Arrays
```javascript
// ============= MEDIAN OF TWO SORTED ARRAYS =============
// Time: O(log min(m, n)) - binary search on smaller array
// Space: O(1) - only using pointers
// KEY INSIGHT: Find partition point that splits combined arrays in half
function findMedianSortedArrays(nums1, nums2) {
    // OPTIMIZATION: Binary search on smaller array
    // WHY? Reduces search space from O(log m) to O(log min(m,n))
    if (nums1.length > nums2.length) {
        return findMedianSortedArrays(nums2, nums1);
    }
    
    const m = nums1.length;
    const n = nums2.length;
    let low = 0;
    let high = m;  // Partition can be from 0 to m elements from nums1
    
    // Binary search for correct partition
    while (low <= high) {
        // Partition nums1 at partitionX
        const partitionX = Math.floor((low + high) / 2);
        // Partition nums2 to balance total elements
        // +1 handles odd total length (left side gets extra element)
        const partitionY = Math.floor((m + n + 1) / 2) - partitionX;
        
        // Find border elements around partitions
        // Handle edge cases with Infinity
        const maxLeftX = partitionX === 0 ? -Infinity : nums1[partitionX - 1];
        const minRightX = partitionX === m ? Infinity : nums1[partitionX];
        
        const maxLeftY = partitionY === 0 ? -Infinity : nums2[partitionY - 1];
        const minRightY = partitionY === n ? Infinity : nums2[partitionY];
        
        // CHECK VALID PARTITION:
        // All elements on left ≤ all elements on right
        if (maxLeftX <= minRightY && maxLeftY <= minRightX) {
            // FOUND! Calculate median
            if ((m + n) % 2 === 0) {
                // Even total: median is average of middle two
                // Max of left partition and min of right partition
                return (Math.max(maxLeftX, maxLeftY) + 
                        Math.min(minRightX, minRightY)) / 2;
            } else {
                // Odd total: median is middle element
                // Which is max of left partition (has extra element)
                return Math.max(maxLeftX, maxLeftY);
            }
        } else if (maxLeftX > minRightY) {
            // nums1's partition is too far right
            // Move partition left in nums1
            high = partitionX - 1;
        } else {
            // nums1's partition is too far left
            // Move partition right in nums1
            low = partitionX + 1;
        }
    }
    
    throw new Error("Arrays not sorted");
}

// EXAMPLE: nums1 = [1,3], nums2 = [2]
// Total length = 3 (odd), median at index 1
// Try partitionX = 1:
// - Left: nums1[0]=1, nums2[0]=2 → max=2
// - Right: nums1[1]=3, nums2[none] → min=3
// Valid partition! Median = max(left) = 2

// VISUALIZATION:
// nums1: [1, 3, | 8, 9]     partitionX = 2
// nums2: [2, 4, 5, | 6, 7]  partitionY = 3
// Combined: [1, 2, 3, 4, 5 | 6, 7, 8, 9]
// Median = (5 + 6) / 2 = 5.5
```

## Interview Tips

### Common Questions
1. **"Explain the recurrence relation for divide and conquer"**
   - T(n) = aT(n/b) + f(n)
   - Use Master Theorem to find complexity
   - a = number of subproblems
   - b = size reduction factor

2. **"When is divide and conquer efficient?"**
   - When subproblems are independent
   - When combine step is efficient
   - When problem exhibits optimal substructure

3. **"Compare recursive vs iterative approaches"**
   - Recursive: cleaner code, stack overhead
   - Iterative: better space complexity
   - Tail recursion can be optimized

### Red Flags to Avoid
- ❌ Not handling base cases properly
- ❌ Inefficient combine step negating benefits
- ❌ Stack overflow on large inputs
- ❌ Not considering parallelization opportunities

### AWS-Specific Follow-ups
- "How does EMR implement MapReduce?"
- "Design a distributed sorting system"
- "Optimize divide-and-conquer for Lambda"
- "Implement parallel processing on EC2 cluster"

## Key Takeaways
1. **Divide and conquer** breaks problems into manageable subproblems
2. **Master Theorem** helps analyze time complexity
3. **Parallelization** is natural with independent subproblems
4. **MapReduce** extends divide-and-conquer to distributed systems
5. **AWS EMR** provides managed infrastructure for large-scale processing

---

*Previous: [← Searching and Sorting](./05-searching-and-sorting.md)*
*Next: [Graph Algorithms →](./07-graph-algorithms.md)*
