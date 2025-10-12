# 📚 Stacks & Queues

## Overview
Stacks and Queues are fundamental linear data structures that control the order of element access. Stacks follow LIFO (Last In, First Out) while Queues follow FIFO (First In, First Out).

## Key Concepts

### Stack Operations
| Operation | Time | Description |
|-----------|------|-------------|
| Push | O(1) | Add element to top |
| Pop | O(1) | Remove top element |
| Peek/Top | O(1) | View top element |
| isEmpty | O(1) | Check if empty |

### Queue Operations
| Operation | Time | Description |
|-----------|------|-------------|
| Enqueue | O(1) | Add to rear |
| Dequeue | O(1) | Remove from front |
| Front | O(1) | View front element |
| isEmpty | O(1) | Check if empty |

## Common Patterns

### Stack Patterns
1. **Matching Brackets** - Validate nested structures
2. **Monotonic Stack** - Track next greater/smaller elements
3. **Expression Evaluation** - Infix, postfix, prefix
4. **Backtracking Helper** - Store states for reversal

### Queue Patterns
1. **BFS Traversal** - Level-order processing
2. **Sliding Window** - Fixed-size window operations
3. **Task Scheduling** - Process in order
4. **Rate Limiting** - Time-based processing

## Problems in This Section

### Stack Problems
- [Valid Parentheses](problems/valid-parentheses.md) - Classic bracket matching
- [Min Stack](problems/min-stack.md) - O(1) minimum retrieval
- [Daily Temperatures](problems/daily-temperatures.md) - Monotonic stack
- [Evaluate RPN](problems/evaluate-rpn.md) - Expression evaluation

### Queue Problems
- [Implement Queue Using Stacks](problems/implement-queue-using-stacks.md) - Two-stack queue
- [Task Scheduler](problems/task-scheduler.md) - CPU scheduling

### Mixed Problems
- [Waiter](problems/waiter.md) - Multiple stacks manipulation

## Implementation Templates

```javascript
// Stack Implementation
class Stack {
    constructor() {
        this.items = [];
    }
    
    push(item) {
        this.items.push(item);
    }
    
    pop() {
        return this.items.pop();
    }
    
    peek() {
        return this.items[this.items.length - 1];
    }
    
    isEmpty() {
        return this.items.length === 0;
    }
}

// Queue Implementation
class Queue {
    constructor() {
        this.items = [];
    }
    
    enqueue(item) {
        this.items.push(item);
    }
    
    dequeue() {
        return this.items.shift();
    }
    
    front() {
        return this.items[0];
    }
    
    isEmpty() {
        return this.items.length === 0;
    }
}

// Deque (Double-ended queue)
class Deque {
    constructor() {
        this.items = [];
    }
    
    addFront(item) {
        this.items.unshift(item);
    }
    
    addRear(item) {
        this.items.push(item);
    }
    
    removeFront() {
        return this.items.shift();
    }
    
    removeRear() {
        return this.items.pop();
    }
}
```

## Advanced Techniques

### Monotonic Stack
```javascript
// Find next greater element
function nextGreater(nums) {
    const stack = [];
    const result = new Array(nums.length).fill(-1);
    
    for (let i = 0; i < nums.length; i++) {
        while (stack.length && nums[stack[stack.length - 1]] < nums[i]) {
            const idx = stack.pop();
            result[idx] = nums[i];
        }
        stack.push(i);
    }
    
    return result;
}
```

### Circular Queue
```javascript
class CircularQueue {
    constructor(k) {
        this.data = new Array(k);
        this.head = 0;
        this.tail = 0;
        this.size = 0;
        this.capacity = k;
    }
    
    enqueue(value) {
        if (this.size === this.capacity) return false;
        this.data[this.tail] = value;
        this.tail = (this.tail + 1) % this.capacity;
        this.size++;
        return true;
    }
}
```

## Tips for Success
1. **Visualize the operations** - Draw the stack/queue state changes
2. **Consider edge cases** - Empty structure, single element
3. **Use built-in arrays** - JavaScript arrays work well for both
4. **Think about order** - What goes in first, what comes out first?
5. **Multiple stacks/queues** - Some problems need auxiliary structures

## When to Use

### Use Stack When:
✅ Need LIFO ordering
✅ Matching nested structures
✅ Tracking history for undo
✅ DFS traversal
✅ Expression evaluation

### Use Queue When:
✅ Need FIFO ordering
✅ BFS traversal
✅ Processing in order received
✅ Buffering data streams
✅ Task scheduling
