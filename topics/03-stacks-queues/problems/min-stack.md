# Min Stack

## 🎯 What Are We Trying To Solve?

Creating a stack that can retrieve the minimum element in constant time is like having a stack of books where you always know which one is the thinnest without searching through them all.

## 📝 The Problem

Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

```
Example:
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin(); // Returns -3
minStack.pop();
minStack.top();    // Returns 0
minStack.getMin(); // Returns -2
```

## 💻 The Code

```javascript
/**
 * Min Stack - Two Stacks Approach
 * All operations: O(1) time
 */
class MinStack {
    constructor() {
        this.stack = [];
        this.minStack = [];
    }
    
    push(val) {
        this.stack.push(val);
        
        // Push to minStack if it's empty or val is <= current min
        if (this.minStack.length === 0 || val <= this.getMin()) {
            this.minStack.push(val);
        }
    }
    
    pop() {
        const popped = this.stack.pop();
        
        // If popped value is the min, remove from minStack
        if (popped === this.getMin()) {
            this.minStack.pop();
        }
        
        return popped;
    }
    
    top() {
        return this.stack[this.stack.length - 1];
    }
    
    getMin() {
        return this.minStack[this.minStack.length - 1];
    }
}

/**
 * Single Stack with Pairs
 */
class MinStackPairs {
    constructor() {
        this.stack = []; // Each element: [value, minSoFar]
    }
    
    push(val) {
        const min = this.stack.length === 0 ? val : 
                    Math.min(val, this.getMin());
        this.stack.push([val, min]);
    }
    
    pop() {
        return this.stack.pop()[0];
    }
    
    top() {
        return this.stack[this.stack.length - 1][0];
    }
    
    getMin() {
        return this.stack[this.stack.length - 1][1];
    }
}
```

## 🎨 Key Insights

- **Auxiliary Stack**: Track minimums separately
- **Pair Storage**: Store value with min-so-far
- **Constant Time**: All operations remain O(1)

## ⏱️ Complexity

- Time: O(1) for all operations
- Space: O(n) for storing elements
