# 🔄 Recursion

## Overview
Recursion is a problem-solving technique where a function calls itself to solve smaller instances of the same problem. It's the foundation for many algorithms including divide-and-conquer, dynamic programming, and tree/graph traversals.

## Key Concepts

### Anatomy of Recursion
1. **Base Case**: Termination condition
2. **Recursive Case**: Problem reduction
3. **Call Stack**: Execution context

### Types of Recursion
- **Linear Recursion**: Single recursive call
- **Binary Recursion**: Two recursive calls (trees)
- **Multiple Recursion**: Multiple calls (n-queens)
- **Tail Recursion**: Last operation is recursive call
- **Mutual Recursion**: Functions call each other

## Common Patterns

### Basic Template
```javascript
function recursiveSolution(input) {
    // Base case(s)
    if (isBaseCase(input)) {
        return baseValue;
    }
    
    // Recursive case
    // 1. Break down problem
    const smaller = makeSmaller(input);
    
    // 2. Recursive call
    const result = recursiveSolution(smaller);
    
    // 3. Combine results
    return combine(result, currentWork);
}
```

### Divide and Conquer
```javascript
function divideAndConquer(arr, low, high) {
    // Base case
    if (low >= high) {
        return baseSolution(arr[low]);
    }
    
    // Divide
    const mid = Math.floor((low + high) / 2);
    
    // Conquer
    const left = divideAndConquer(arr, low, mid);
    const right = divideAndConquer(arr, mid + 1, high);
    
    // Combine
    return merge(left, right);
}
```

## Problems in This Section

### Basic
- [Recursion Fundamentals](problems/recursion-fundamentals.md) - Fibonacci & Print 1 to N

### Advanced
- [Count Inversions](problems/count-inversions.md) - Merge sort application
- [Median of Two Sorted Arrays](problems/median-of-two-sorted-arrays.md) - Binary search recursion

## Classic Recursion Problems

### Factorial
```javascript
function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
// Iterative is often better for this
```

### Power Function
```javascript
function power(base, exp) {
    if (exp === 0) return 1;
    
    if (exp % 2 === 0) {
        const half = power(base, exp / 2);
        return half * half;
    } else {
        return base * power(base, exp - 1);
    }
}
// O(log n) time complexity
```

### Permutations
```javascript
function permutations(arr) {
    if (arr.length <= 1) return [arr];
    
    const result = [];
    
    for (let i = 0; i < arr.length; i++) {
        const current = arr[i];
        const remaining = [...arr.slice(0, i), ...arr.slice(i + 1)];
        const perms = permutations(remaining);
        
        for (const perm of perms) {
            result.push([current, ...perm]);
        }
    }
    
    return result;
}
```

### Backtracking Template
```javascript
function backtrack(state, choices, result) {
    // Base case: found solution
    if (isSolution(state)) {
        result.push([...state]);
        return;
    }
    
    // Try each choice
    for (const choice of choices) {
        if (isValid(state, choice)) {
            // Make choice
            state.push(choice);
            
            // Recurse
            backtrack(state, choices, result);
            
            // Undo choice (backtrack)
            state.pop();
        }
    }
}
```

## Recursion vs Iteration

### When to Use Recursion
✅ **Good for:**
- Tree/graph traversal
- Divide and conquer
- Backtracking problems
- Problems with recursive structure
- Clean, readable code

❌ **Avoid when:**
- Deep recursion (stack overflow)
- Simple iteration works
- Performance critical
- Tail recursion not optimized

### Converting to Iteration
```javascript
// Recursive
function sumRecursive(n) {
    if (n <= 0) return 0;
    return n + sumRecursive(n - 1);
}

// Iterative
function sumIterative(n) {
    let sum = 0;
    for (let i = 1; i <= n; i++) {
        sum += i;
    }
    return sum;
}

// Using explicit stack
function treeTraversalIterative(root) {
    const stack = [root];
    const result = [];
    
    while (stack.length) {
        const node = stack.pop();
        if (!node) continue;
        
        result.push(node.val);
        stack.push(node.right);
        stack.push(node.left);
    }
    
    return result;
}
```

## Optimization Techniques

### Memoization
```javascript
function fibMemo(n, memo = {}) {
    if (n in memo) return memo[n];
    if (n <= 1) return n;
    
    memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    return memo[n];
}
```

### Tail Call Optimization
```javascript
// Not tail recursive
function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1); // Multiplication after return
}

// Tail recursive
function factorialTail(n, acc = 1) {
    if (n <= 1) return acc;
    return factorialTail(n - 1, n * acc); // Nothing after recursive call
}
```

## Common Pitfalls

1. **Missing Base Case** - Infinite recursion
2. **Stack Overflow** - Too deep recursion
3. **Redundant Calculations** - Not memoizing
4. **Wrong Problem Division** - Not getting smaller
5. **State Mutation** - Side effects in recursion

## Tips for Success

1. **Identify base cases first** - When to stop
2. **Trust the recursion** - Assume it works for smaller inputs
3. **Draw the recursion tree** - Visualize calls
4. **Think about the call stack** - Understand execution order
5. **Consider memoization** - Avoid repeated work
6. **Test with small inputs** - Debug easier

## Interview Approach

1. Can I break this into smaller subproblems?
2. What's my base case?
3. How do I combine results?
4. Do I need to track state?
5. Should I use memoization?
6. Would iteration be simpler?

## Classic Interview Problems

- Generate all subsets
- N-Queens problem
- Sudoku solver
- Word search in grid
- Regular expression matching
- Decode ways
- Restore IP addresses
