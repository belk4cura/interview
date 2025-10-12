# Sliding Window Maximum

## 🎯 What Are We Trying To Solve?

Finding the maximum element in every window of size k as it slides through an array is like tracking the tallest person visible through a moving window. We need an efficient way to update our answer as the window moves.

## 📝 The Problem

Given an array and a window size k, return an array of the maximum values for each window position as it slides from left to right.

```
Example:
nums = [1,3,-1,-3,5,3,6,7], k = 3

Window position                Max
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7

Output: [3,3,5,5,6,7]
```

## 💻 The Code

```javascript
/**
 * Sliding Window Maximum - Deque Solution
 * Time: O(n), Space: O(k)
 */
function maxSlidingWindow(nums, k) {
    const result = [];
    const deque = []; // Store indices, not values
    
    for (let i = 0; i < nums.length; i++) {
        // Remove indices outside current window
        while (deque.length && deque[0] <= i - k) {
            deque.shift();
        }
        
        // Remove indices of smaller elements (they'll never be max)
        while (deque.length && nums[deque[deque.length - 1]] < nums[i]) {
            deque.pop();
        }
        
        // Add current index
        deque.push(i);
        
        // Add max to result (front of deque) if window is complete
        if (i >= k - 1) {
            result.push(nums[deque[0]]);
        }
    }
    
    return result;
}

// Alternative: Using heap (less efficient)
function maxSlidingWindowHeap(nums, k) {
    const result = [];
    const heap = new MaxHeap();
    
    // Process first window
    for (let i = 0; i < k; i++) {
        heap.insert({val: nums[i], index: i});
    }
    result.push(heap.peek().val);
    
    // Slide window
    for (let i = k; i < nums.length; i++) {
        heap.insert({val: nums[i], index: i});
        
        // Remove elements outside window
        while (heap.peek().index <= i - k) {
            heap.extractMax();
        }
        
        result.push(heap.peek().val);
    }
    
    return result;
}
```

## 🎨 Key Insights

- **Deque (Double-ended Queue)**: Perfect for maintaining window maximum
- **Monotonic Decreasing**: Keep elements in decreasing order
- **Index Storage**: Store indices to track window boundaries

## ⏱️ Complexity

- Time: O(n) - Each element enters/exits deque once
- Space: O(k) - Deque size bounded by window size
