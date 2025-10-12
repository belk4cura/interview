# What's the Complexity?

## 🎯 What Are We Trying To Solve?

You're given code snippets and need to figure out how their performance changes as the input grows. Think of it like predicting traffic: will adding more cars cause a smooth flow (linear), gridlock (quadratic), or something in between?

## 📝 The Problem

Analyze this code and determine its time complexity:

```javascript
function mystery1(n) {
    let count = 0;
    for (let i = n; i > 0; i = Math.floor(i / 2)) {
        count++;
    }
    return count;
}
```

## 🧠 How to Think About This

Let's trace through with actual numbers:

```
n = 32:
Step 1: i = 32 → count = 1
Step 2: i = 16 → count = 2  
Step 3: i = 8  → count = 3
Step 4: i = 4  → count = 4
Step 5: i = 2  → count = 5
Step 6: i = 1  → count = 6
Step 7: i = 0  → STOP

Total steps: 6
```

Notice the pattern? We're dividing by 2 each time!

### Visual Pattern
```
32 → 16 → 8 → 4 → 2 → 1
   ÷2   ÷2  ÷2  ÷2  ÷2

This is like zooming out on a map:
🏠 (1 house)
🏠🏠 (2 houses) 
🏠🏠🏠🏠 (4 houses)
🏠🏠🏠🏠🏠🏠🏠🏠 (8 houses)
...
Each zoom level shows double the area!
```

## 💡 The Answer

**Time Complexity: O(log n)**

### Why O(log n)?
- We divide `i` by 2 in each iteration
- The question becomes: "How many times can we divide n by 2 until we reach 1?"
- The answer is: log₂(n)

### Real-World Analogy
It's like finding a word in a dictionary:
1. Open to the middle
2. Is your word before or after?
3. Go to the middle of that half
4. Repeat until found

Each step eliminates half the remaining pages!

## 📊 Let's Count Steps for Different Inputs

```
n        Steps    log₂(n)
1        1        0
2        2        1
4        3        2
8        4        3
16       5        4
32       6        5
64       7        6
128      8        7
1024     11       10
1,000,000  ~21    ~20
```

Notice: Even when n gets HUGE (1 million), we only need about 20 steps!

## 🎨 Step-by-Step Visualization

```javascript
function mystery1(n) {
    let count = 0;
    // Loop condition: keep going while i > 0
    for (let i = n; i > 0; i = Math.floor(i / 2)) {
        count++;
        console.log(`Step ${count}: i = ${i}`);
    }
    return count;
}

// Running mystery1(20):
// Step 1: i = 20
// Step 2: i = 10  
// Step 3: i = 5
// Step 4: i = 2
// Step 5: i = 1
// Returns: 5
```

## 🔄 Similar Patterns to Recognize

### Pattern 1: Dividing by Constant
```javascript
// Also O(log n) - dividing by 3
for (let i = n; i > 0; i = Math.floor(i / 3)) {
    // ...
}
```

### Pattern 2: Multiplying Until Limit
```javascript
// Also O(log n) - multiplying by 2
for (let i = 1; i < n; i = i * 2) {
    // ...
}
```

### Pattern 3: Binary Search
```javascript
function binarySearch(arr, target) {
    let left = 0, right = arr.length - 1;
    
    while (left <= right) {
        let mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) return mid;
        
        // Eliminate half
        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
// Time: O(log n) - cutting search space in half each time
```

## 🚫 Common Mistakes

1. **Confusing with O(n)**
   ```javascript
   // This is O(n), not O(log n)!
   for (let i = n; i > 0; i--) {  // Subtracting, not dividing
       count++;
   }
   ```

2. **Missing the Division Pattern**
   - Look for `i = i / 2` or `i = Math.floor(i / 2)`
   - This is the key indicator of logarithmic time

3. **Not Understanding Logarithms**
   - log₂(n) asks: "2 to what power equals n?"
   - log₂(8) = 3 because 2³ = 8
   - log₂(32) = 5 because 2⁵ = 32

## 📈 Performance in Practice

```
If one operation takes 1 microsecond:

n = 10:        ~3 μs    (instant)
n = 1,000:     ~10 μs   (instant)  
n = 1,000,000: ~20 μs   (instant)
n = 1 billion: ~30 μs   (instant)

Compare to O(n):
n = 1 billion: 1,000,000 μs = 1 second!
```

## 🔗 Related Problems

- Binary Search implementation
- Finding height of a balanced tree
- Counting bits in binary representation
- Finding power of 2

## 💡 Key Takeaway

**O(log n) is FAST!** When you see division by a constant (especially 2) in a loop, think logarithmic. This is why binary search is so powerful - even with a billion elements, you only need about 30 comparisons!

Remember: 
- Dividing repeatedly = O(log n)
- Subtracting repeatedly = O(n)
- Nested loops = O(n²)
