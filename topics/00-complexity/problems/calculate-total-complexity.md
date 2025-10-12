# Calculate Total Complexity

## 🎯 What Are We Trying To Solve?

Real code rarely does just one thing. It might sort data, then search it, then process results. How do we figure out the total complexity? It's like calculating travel time when you walk, then drive, then walk again - we need to understand which part dominates!

## 📝 The Problem

What's the total time complexity of this function?

```javascript
function complexFunction(arr) {
    // Part A: Sort the array
    arr.sort();  
    
    // Part B: Print each element
    for (let i = 0; i < arr.length; i++) {
        console.log(arr[i]);
    }
    
    // Part C: Print first 100 numbers
    for (let i = 0; i < 100; i++) {
        console.log(i);
    }
    
    // Part D: Print each element 5 times
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < 5; j++) {
            console.log(arr[i]);
        }
    }
}
```

## 🧠 Breaking It Down Step by Step

Let's analyze each part separately first:

### Part A: Sorting
```javascript
arr.sort();  // JavaScript's sort is typically O(n log n)
```
```
For n = 1000:
Operations ≈ 1000 × log(1000) ≈ 1000 × 10 = 10,000
```

### Part B: Single Loop
```javascript
for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);  // O(1) operation
}
```
```
For n = 1000:
Operations = 1000
```

### Part C: Constant Loop
```javascript
for (let i = 0; i < 100; i++) {  // Always 100, regardless of n
    console.log(i);
}
```
```
For any n:
Operations = 100 (constant!)
```

### Part D: Nested Loop (with constant inner)
```javascript
for (let i = 0; i < arr.length; i++) {      // n times
    for (let j = 0; j < 5; j++) {           // 5 times (constant!)
        console.log(arr[i]);
    }
}
```
```
For n = 1000:
Operations = 1000 × 5 = 5000
But 5 is constant, so this is O(n)
```

## 📊 Combining Complexities

Now let's add them all up:

```
Part A: O(n log n)
Part B: O(n)
Part C: O(1)
Part D: O(n)

Total: O(n log n) + O(n) + O(1) + O(n)
```

### Visual Comparison
```
For n = 1000:

Part A (n log n): ████████████████████ 10,000 operations
Part B (n):       ██                    1,000 operations
Part C (1):       ▌                       100 operations
Part D (n):       █████                 5,000 operations

Total:            ████████████████████ ~16,100 operations
```

## 💡 The Answer

**Total Complexity: O(n log n)**

### Why Not O(n log n + 2n + 1)?

Big O follows these rules:

#### Rule 1: Keep Only the Dominant Term
```
As n grows:
n log n grows much faster than n
n grows much faster than 1

So: O(n log n + n + n + 1) → O(n log n)
```

#### Visual Growth Comparison
```
n        n log n    2n      1      Winner
10       30         20      1      n log n
100      700        200     1      n log n
1000     10,000     2000    1      n log n
10000    140,000    20000   1      n log n

The gap keeps growing!
```

## 🎨 Real-World Analogy

Imagine your morning routine:

```
1. Drive to gym: 30 minutes (dominates)
2. Walk from parking: 2 minutes
3. Tie shoes: 30 seconds
4. Walk to equipment: 1 minute

Total time? "About 30 minutes" - we ignore the small stuff!
```

Similarly:
```
O(n²) + O(n) + O(1) = O(n²)     "The n² dominates"
O(n log n) + O(n) = O(n log n)  "The n log n dominates"
O(n) + O(n) = O(2n) = O(n)      "Constants don't matter"
```

## 📈 Dominance Hierarchy

Which complexity "wins" when combined?

```
O(n!) > O(2ⁿ) > O(n³) > O(n²) > O(n log n) > O(n) > O(log n) > O(1)
↑                                                                    ↑
Always dominates                                          Always loses
```

### Quick Reference
```javascript
// O(n²) - The n² dominates everything else
function example1(arr) {
    arr.sort();           // O(n log n)
    for (i of arr) {
        for (j of arr) {  // O(n²)
            process();
        }
    }
}

// O(n) - All parts are linear, so they add to O(n)
function example2(arr) {
    for (item of arr) {}  // O(n)
    for (item of arr) {}  // O(n)
    for (item of arr) {}  // O(n)
    // Total: O(3n) = O(n)
}

// O(n log n) - Sorting dominates
function example3(arr) {
    arr.sort();           // O(n log n)
    return arr[0];        // O(1)
}
```

## 🚫 Common Mistakes

1. **Adding Instead of Taking Maximum**
   ```javascript
   // Wrong: "It's O(n + n log n + 1)"
   // Right: "It's O(n log n)"
   ```

2. **Keeping Constants**
   ```javascript
   // Wrong: "The loop runs 5n times so it's O(5n)"
   // Right: "It's O(n)"
   ```

3. **Missing Hidden Complexity**
   ```javascript
   for (let item of arr) {
       arr.includes(item);  // This is O(n)!
   }
   // Total: O(n²), not O(n)
   ```

## 🔑 Pro Tips

### Tip 1: Look for the Bottleneck
The slowest part determines overall speed, like the narrowest part of a pipe limits water flow.

### Tip 2: When in Doubt, Count
```javascript
n = 10:
Part A: 10 × 3 = 30
Part B: 10
Part C: 100  // Uh oh, constant is bigger than n here!
Part D: 50

// But as n grows...
n = 1,000,000:
Part A: 20,000,000  ← This dominates!
Part B: 1,000,000
Part C: 100
Part D: 5,000,000
```

### Tip 3: Multiple Variables
```javascript
function process(arr1, arr2) {
    for (let item of arr1) {}    // O(n)
    for (let item of arr2) {}    // O(m)
}
// Total: O(n + m) - Can't simplify without knowing relationship
```

## 💡 Key Takeaway

**Always identify and keep the dominant term!** In practice:
1. Find the complexity of each section
2. Identify the highest-order term
3. Drop constants and smaller terms
4. That's your answer!

Remember: Big O is about the big picture as n approaches infinity. The details get lost in the noise!
