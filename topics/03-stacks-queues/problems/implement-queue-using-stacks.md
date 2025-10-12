# Implement Queue Using Stacks

## 🎯 What Are We Trying To Solve?

Imagine you have two stacks of plates but need to serve customers in the order they arrived (first come, first served). You need to cleverly use your two stacks to simulate a queue's behavior. It's like having two "last in, first out" structures work together to create a "first in, first out" behavior!

## 📝 The Problem

Implement a queue using only two stacks. The queue should support:
- `push(x)`: Add element x to the back of the queue
- `pop()`: Remove and return the element from the front
- `peek()`: Return the front element without removing it
- `empty()`: Return whether the queue is empty

You can only use standard stack operations: push to top, peek/pop from top, size, and is empty.

## 🧠 How to Think About This

Think of it like transferring water between two buckets:
- **Input Stack**: Where new elements arrive (like pouring water in)
- **Output Stack**: Where elements leave from (like pouring water out)

When the output stack is empty and we need to dequeue, we transfer everything from input to output, reversing the order!

### Visual Process
```
Enqueue 1, 2, 3:
Input:  [3, 2, 1] ← top
Output: []

Dequeue:
Transfer all → Input: []
               Output: [1, 2, 3] ← top
Pop from output → Returns 1
```

## 💻 The Code

```javascript
/**
 * Queue implementation using two stacks
 * Push: O(1)
 * Pop/Peek: Amortized O(1), Worst case O(n)
 */
class MyQueue {
    constructor() {
        this.inStack = [];   // For enqueue operations
        this.outStack = [];  // For dequeue operations
    }
    
    /**
     * Add element to back of queue
     * Time: O(1) - Just push to input stack
     */
    push(x) {
        this.inStack.push(x);
    }
    
    /**
     * Remove and return front element
     * Time: Amortized O(1)
     */
    pop() {
        // Ensure outStack has elements
        this.peek();
        return this.outStack.pop();
    }
    
    /**
     * Get front element without removing
     * Time: Amortized O(1)
     */
    peek() {
        // If output stack is empty, transfer from input
        if (this.outStack.length === 0) {
            // Transfer ALL elements from input to output
            // This reverses the order, making FIFO work!
            while (this.inStack.length > 0) {
                this.outStack.push(this.inStack.pop());
            }
        }
        
        // Front of queue is now at top of output stack
        return this.outStack[this.outStack.length - 1];
    }
    
    /**
     * Check if queue is empty
     * Time: O(1)
     */
    empty() {
        return this.inStack.length === 0 && this.outStack.length === 0;
    }
}

// Test the queue
const queue = new MyQueue();
queue.push(1);
queue.push(2);
queue.push(3);
console.log(queue.peek());  // 1 (front of queue)
console.log(queue.pop());   // 1 (removed)
queue.push(4);
console.log(queue.pop());   // 2
console.log(queue.pop());   // 3
console.log(queue.pop());   // 4
console.log(queue.empty()); // true
```

## 🎨 Step-by-Step Walkthrough

Let's trace through a series of operations:

```
Operation: push(1), push(2), push(3), pop(), push(4), pop()

Initial state:
inStack: []
outStack: []

Step 1: push(1)
- Add to inStack
- inStack: [1]
- outStack: []

Step 2: push(2)
- Add to inStack
- inStack: [1, 2]
- outStack: []

Step 3: push(3)
- Add to inStack
- inStack: [1, 2, 3]
- outStack: []

Step 4: pop()
- outStack is empty, transfer all from inStack
- Transfer: 3 → outStack: [3]
- Transfer: 2 → outStack: [3, 2]
- Transfer: 1 → outStack: [3, 2, 1]
- inStack: []
- Pop from outStack: returns 1
- outStack: [3, 2]

Step 5: push(4)
- Add to inStack
- inStack: [4]
- outStack: [3, 2]

Step 6: pop()
- outStack not empty, pop directly
- Returns: 2
- inStack: [4]
- outStack: [3]
```

## 📊 Why Amortized O(1)?

### The Key Insight
Each element moves at most twice:
1. Once from input to output stack
2. Once out of the output stack

```
Element Journey:
push → inStack → transfer → outStack → pop
     ↑         ↑          ↑          ↑
    O(1)      O(1)       O(1)       O(1)

Total per element: O(1) amortized
```

### Visual Proof
```
Scenario: Push n elements, then pop all

Operations:
Push 1, Push 2, ..., Push n → n × O(1) = O(n)
Pop (triggers transfer) → O(n) for transfer + O(1) for pop
Pop, Pop, ... (n-1 times) → (n-1) × O(1) = O(n-1)

Total: O(n) + O(n) + O(n-1) = O(n)
Operations: 2n
Average per operation: O(n) / 2n = O(1)
```

## 🚫 Common Mistakes

1. **Transferring on Every Operation**
   ```javascript
   // Wrong: Inefficient, transfers unnecessarily
   pop() {
       while (this.inStack.length > 0) {
           this.outStack.push(this.inStack.pop());
       }
       const result = this.outStack.pop();
       while (this.outStack.length > 0) {
           this.inStack.push(this.outStack.pop());
       }
       return result;
   }
   // This is O(n) per operation!
   
   // Right: Transfer only when needed
   if (this.outStack.length === 0) {
       // Transfer once, use multiple times
   }
   ```

2. **Partial Transfers**
   ```javascript
   // Wrong: Only transfer one element
   if (this.outStack.length === 0) {
       this.outStack.push(this.inStack.pop());
   }
   
   // Right: Transfer ALL elements
   while (this.inStack.length > 0) {
       this.outStack.push(this.inStack.pop());
   }
   ```

3. **Checking Wrong Stack for Empty**
   ```javascript
   // Wrong: Only checking one stack
   empty() {
       return this.inStack.length === 0;
   }
   
   // Right: Check both stacks
   empty() {
       return this.inStack.length === 0 && this.outStack.length === 0;
   }
   ```

## 🔄 Alternative: Single Transfer Point

```javascript
// Alternative: Always transfer before pop/peek
class MyQueueAlternative {
    constructor() {
        this.stack1 = [];
        this.stack2 = [];
    }
    
    push(x) {
        // Always push to stack1
        this.stack1.push(x);
    }
    
    pop() {
        this.transferIfNeeded();
        return this.stack2.pop();
    }
    
    peek() {
        this.transferIfNeeded();
        return this.stack2[this.stack2.length - 1];
    }
    
    transferIfNeeded() {
        // Only transfer when stack2 is empty
        if (this.stack2.length === 0) {
            while (this.stack1.length > 0) {
                this.stack2.push(this.stack1.pop());
            }
        }
    }
    
    empty() {
        return this.stack1.length === 0 && this.stack2.length === 0;
    }
}
```

## 🔗 Related Problems

- **Implement Stack Using Queues** - The reverse problem
- **Design Circular Queue** - Fixed-size queue implementation
- **Moving Average from Data Stream** - Queue with size limit
- **Design Hit Counter** - Queue with time-based eviction

## 💡 Key Takeaways

1. **Two Stacks Create FIFO**: Reversing twice gives original order
2. **Lazy Transfer is Efficient**: Transfer only when necessary
3. **Amortized Analysis**: Each element moves at most twice total
4. **State Split**: Input and output stacks serve different purposes

This technique of using two data structures to simulate another is common in system design and interview problems!
