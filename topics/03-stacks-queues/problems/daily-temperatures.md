# Daily Temperatures

## 🎯 What Are We Trying To Solve?

Imagine you're a weather forecaster looking at a week's temperatures. For each day, you want to tell people how many days they need to wait until a warmer day. It's like standing in line and asking "how many people ahead of me are shorter than me?" - but for temperatures!

## 📝 The Problem

Given an array of daily temperatures, return an array where each element tells you how many days you have to wait after that day to get a warmer temperature. If there's no future day with warmer temperature, put 0.

```
Input:  [73, 74, 75, 71, 69, 72, 76, 73]
Output: [ 1,  1,  4,  2,  1,  1,  0,  0]

Day 0 (73°): Wait 1 day for 74°
Day 1 (74°): Wait 1 day for 75°
Day 2 (75°): Wait 4 days for 76°
Day 3 (71°): Wait 2 days for 72°
...
```

## 🧠 How to Think About This

The key insight: Use a **monotonic decreasing stack** - a stack that maintains temperatures in decreasing order. When we find a warmer day, we can pop all the colder days that came before it!

### Visual Process
```
Stack stores indices, not values!

[73, 74, 75, 71, 69, 72, 76, 73]
 0   1   2   3   4   5   6   7

Process day by day:
Day 0 (73°): Stack: [0]
Day 1 (74°): 74 > 73, pop 0, answer[0] = 1-0 = 1
             Stack: [1]
Day 2 (75°): 75 > 74, pop 1, answer[1] = 2-1 = 1
             Stack: [2]
...
```

## 💻 The Code

```javascript
/**
 * Find days until warmer temperature
 * Time: O(n) - Each element pushed and popped at most once
 * Space: O(n) - Stack can contain all indices in worst case
 */
function dailyTemperatures(temperatures) {
    const n = temperatures.length;
    const result = new Array(n).fill(0);
    const stack = []; // Stack of indices, not temperatures!
    
    for (let i = 0; i < n; i++) {
        // Current temperature
        const currentTemp = temperatures[i];
        
        // Pop all days with lower temperature
        // They've found their warmer day (today!)
        while (stack.length > 0 && 
               temperatures[stack[stack.length - 1]] < currentTemp) {
            const prevIndex = stack.pop();
            result[prevIndex] = i - prevIndex; // Days to wait
        }
        
        // Push current day index to stack
        stack.push(i);
    }
    
    // Remaining indices in stack have no warmer day
    // They're already set to 0 by fill()
    
    return result;
}

// Test cases
console.log(dailyTemperatures([73,74,75,71,69,72,76,73]));
// Output: [1,1,4,2,1,1,0,0]

console.log(dailyTemperatures([30,40,50,60]));
// Output: [1,1,1,0] - Increasing temperatures

console.log(dailyTemperatures([30,60,90]));
// Output: [1,1,0]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `[73, 74, 75, 71, 69, 72, 76, 73]`:

```
Initial: result = [0,0,0,0,0,0,0,0], stack = []

Day 0 (73°):
- Stack empty, push 0
- Stack: [0]
- Result: [0,0,0,0,0,0,0,0]

Day 1 (74°):
- 74 > temps[0]=73, pop 0
- result[0] = 1-0 = 1
- Push 1
- Stack: [1]
- Result: [1,0,0,0,0,0,0,0]

Day 2 (75°):
- 75 > temps[1]=74, pop 1
- result[1] = 2-1 = 1
- Push 2
- Stack: [2]
- Result: [1,1,0,0,0,0,0,0]

Day 3 (71°):
- 71 < temps[2]=75, no popping
- Push 3
- Stack: [2,3]
- Result: [1,1,0,0,0,0,0,0]

Day 4 (69°):
- 69 < temps[3]=71, no popping
- Push 4
- Stack: [2,3,4]
- Result: [1,1,0,0,0,0,0,0]

Day 5 (72°):
- 72 > temps[4]=69, pop 4, result[4] = 5-4 = 1
- 72 > temps[3]=71, pop 3, result[3] = 5-3 = 2
- 72 < temps[2]=75, stop
- Push 5
- Stack: [2,5]
- Result: [1,1,0,2,1,0,0,0]

Day 6 (76°):
- 76 > temps[5]=72, pop 5, result[5] = 6-5 = 1
- 76 > temps[2]=75, pop 2, result[2] = 6-2 = 4
- Push 6
- Stack: [6]
- Result: [1,1,4,2,1,1,0,0]

Day 7 (73°):
- 73 < temps[6]=76, no popping
- Push 7
- Stack: [6,7]
- Result: [1,1,4,2,1,1,0,0]

End: Stack has [6,7] - these days have no warmer future
Final: [1,1,4,2,1,1,0,0] ✓
```

## 📊 Why O(n) Time Complexity?

### The Key Insight
Each index is:
1. Pushed to stack exactly once
2. Popped from stack at most once

```
Total operations = n pushes + (at most) n pops = 2n = O(n)

Even though we have nested loops:
for (each day) {              // n iterations
    while (stack not empty) {  // Each element popped once total
        pop();
    }
}
```

## 🚫 Common Mistakes

1. **Storing Values Instead of Indices**
   ```javascript
   // Wrong: Can't calculate days difference
   stack.push(temperatures[i]);
   
   // Right: Store index to calculate difference
   stack.push(i);
   ```

2. **Using Wrong Comparison**
   ```javascript
   // Wrong: <= would pop equal temperatures
   while (temperatures[stack[stack.length - 1]] <= currentTemp)
   
   // Right: Only pop for strictly warmer days
   while (temperatures[stack[stack.length - 1]] < currentTemp)
   ```

3. **Not Initializing Result Array**
   ```javascript
   // Wrong: undefined values
   const result = new Array(n);
   
   // Right: Initialize with 0s
   const result = new Array(n).fill(0);
   ```

## 🔄 Alternative Approaches

### Approach 2: Brute Force (Less Efficient)
```javascript
// O(n²) - Check every future day for each day
function dailyTemperaturesBrute(temperatures) {
    const n = temperatures.length;
    const result = new Array(n).fill(0);
    
    for (let i = 0; i < n; i++) {
        for (let j = i + 1; j < n; j++) {
            if (temperatures[j] > temperatures[i]) {
                result[i] = j - i;
                break; // Found first warmer day
            }
        }
    }
    
    return result;
}
```

### Approach 3: Traverse from Right
```javascript
// Process from right to left
function dailyTemperaturesReverse(temperatures) {
    const n = temperatures.length;
    const result = new Array(n).fill(0);
    const stack = [];
    
    // Process from right to left
    for (let i = n - 1; i >= 0; i--) {
        // Remove temperatures <= current
        while (stack.length > 0 && 
               temperatures[stack[stack.length - 1]] <= temperatures[i]) {
            stack.pop();
        }
        
        // If stack not empty, top has next warmer day
        if (stack.length > 0) {
            result[i] = stack[stack.length - 1] - i;
        }
        
        stack.push(i);
    }
    
    return result;
}
```

## 🔑 Monotonic Stack Pattern

This problem demonstrates the **monotonic stack** pattern:

```javascript
// Template for "next greater element" problems
function nextGreater(arr) {
    const stack = [];
    const result = new Array(arr.length);
    
    for (let i = 0; i < arr.length; i++) {
        // Pop smaller elements
        while (stack.length && arr[stack[stack.length-1]] < arr[i]) {
            const idx = stack.pop();
            result[idx] = i; // Found next greater
        }
        stack.push(i);
    }
    
    return result;
}
```

## 🔗 Related Problems

- **Next Greater Element I & II** - Find next greater in array
- **Largest Rectangle in Histogram** - Monotonic stack for area
- **Trapping Rain Water** - Stack for water levels
- **Remove K Digits** - Monotonic stack for smallest number

## 💡 Key Takeaways

1. **Monotonic Stack**: Maintains elements in sorted order
2. **Store Indices**: Need position info for calculating differences
3. **One Pass Solution**: O(n) despite nested loops
4. **Pattern Recognition**: "Next greater/smaller" → Think monotonic stack

This pattern is extremely powerful for solving "next greater/smaller element" problems efficiently!
