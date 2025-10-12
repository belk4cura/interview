# Recursion Fundamentals (Fibonacci & Print 1 to N)

## 🎯 What Are We Trying To Solve?

Recursion is like Russian nesting dolls - each problem contains a smaller version of itself. We'll explore two classic problems: Fibonacci sequence (where each number is the sum of the previous two) and printing numbers from 1 to N (demonstrating the call stack behavior).

## 📝 The Problems

### Problem 1: Fibonacci Sequence
Calculate the nth Fibonacci number where:
- F(0) = 0
- F(1) = 1
- F(n) = F(n-1) + F(n-2) for n > 1

### Problem 2: Print 1 to N Recursively
Print all numbers from 1 to N using recursion, demonstrating how the call stack unwinds.

```
Examples:
Fibonacci(5) = 5 (sequence: 0, 1, 1, 2, 3, 5)
Print(5) outputs: 1 2 3 4 5
```

## 💻 The Code

```javascript
/**
 * FIBONACCI IMPLEMENTATIONS
 */

// Naive recursive approach - O(2^n) time
function fibonacciNaive(n) {
    // Base cases
    if (n === 0) return 0;
    if (n === 1) return 1;
    
    // Recursive case
    return fibonacciNaive(n - 1) + fibonacciNaive(n - 2);
}

// Memoization (Top-down DP) - O(n) time, O(n) space
function fibonacciMemo(n, memo = {}) {
    // Check memo first
    if (n in memo) return memo[n];
    
    // Base cases
    if (n === 0) return 0;
    if (n === 1) return 1;
    
    // Calculate and store in memo
    memo[n] = fibonacciMemo(n - 1, memo) + fibonacciMemo(n - 2, memo);
    return memo[n];
}

// Alternative: Using Map for memoization
class FibonacciCalculator {
    constructor() {
        this.cache = new Map([[0, 0], [1, 1]]);
    }
    
    calculate(n) {
        if (this.cache.has(n)) {
            return this.cache.get(n);
        }
        
        const result = this.calculate(n - 1) + this.calculate(n - 2);
        this.cache.set(n, result);
        return result;
    }
}

// Iterative approach (Bottom-up DP) - O(n) time, O(1) space
function fibonacciIterative(n) {
    if (n === 0) return 0;
    if (n === 1) return 1;
    
    let prev2 = 0;
    let prev1 = 1;
    
    for (let i = 2; i <= n; i++) {
        const current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }
    
    return prev1;
}

// Matrix exponentiation - O(log n) time
function fibonacciMatrix(n) {
    if (n === 0) return 0;
    if (n === 1) return 1;
    
    function multiplyMatrix(a, b) {
        return [
            [a[0][0] * b[0][0] + a[0][1] * b[1][0], 
             a[0][0] * b[0][1] + a[0][1] * b[1][1]],
            [a[1][0] * b[0][0] + a[1][1] * b[1][0], 
             a[1][0] * b[0][1] + a[1][1] * b[1][1]]
        ];
    }
    
    function matrixPower(matrix, n) {
        if (n === 1) return matrix;
        
        if (n % 2 === 0) {
            const half = matrixPower(matrix, n / 2);
            return multiplyMatrix(half, half);
        } else {
            return multiplyMatrix(matrix, matrixPower(matrix, n - 1));
        }
    }
    
    const baseMatrix = [[1, 1], [1, 0]];
    const result = matrixPower(baseMatrix, n);
    return result[0][1];
}

/**
 * PRINT 1 TO N RECURSIVELY
 */

// Print 1 to N (demonstrates forward recursion)
function print1ToN(n) {
    if (n <= 0) return;
    
    // Recursive call first
    print1ToN(n - 1);
    
    // Then print current number
    console.log(n);
}

// Alternative: Collect in array
function collect1ToN(n) {
    if (n <= 0) return [];
    
    const smaller = collect1ToN(n - 1);
    smaller.push(n);
    return smaller;
}

// Print N to 1 (demonstrates backward recursion)
function printNTo1(n) {
    if (n <= 0) return;
    
    // Print first
    console.log(n);
    
    // Then recursive call
    printNTo1(n - 1);
}

// Print with custom format
function printRecursiveFormatted(n, prefix = '') {
    if (n <= 0) {
        console.log(prefix + 'Done!');
        return;
    }
    
    printRecursiveFormatted(n - 1, prefix + '  ');
    console.log(prefix + `Number: ${n}`);
}

/**
 * VARIATIONS AND APPLICATIONS
 */

// Generate Fibonacci sequence up to n
function fibonacciSequence(n) {
    const sequence = [];
    const memo = {};
    
    function fib(i) {
        if (i === 0) return 0;
        if (i === 1) return 1;
        if (memo[i]) return memo[i];
        
        memo[i] = fib(i - 1) + fib(i - 2);
        return memo[i];
    }
    
    for (let i = 0; i <= n; i++) {
        sequence.push(fib(i));
    }
    
    return sequence;
}

// Find index of Fibonacci number
function findFibonacciIndex(target) {
    let a = 0, b = 1;
    let index = 1;
    
    if (target === 0) return 0;
    
    while (b < target) {
        const temp = a + b;
        a = b;
        b = temp;
        index++;
    }
    
    return b === target ? index : -1;
}

// Check if number is Fibonacci
function isFibonacci(n) {
    // A number is Fibonacci if one of (5*n*n + 4) or (5*n*n - 4) is a perfect square
    function isPerfectSquare(x) {
        const sqrt = Math.sqrt(x);
        return sqrt === Math.floor(sqrt);
    }
    
    return isPerfectSquare(5 * n * n + 4) || isPerfectSquare(5 * n * n - 4);
}

// Test cases
console.log("Fibonacci Tests:");
console.log(fibonacciNaive(10));      // 55
console.log(fibonacciMemo(10));       // 55
console.log(fibonacciIterative(10));  // 55
console.log(fibonacciSequence(10));   // [0,1,1,2,3,5,8,13,21,34,55]

console.log("\nPrint 1 to N Tests:");
print1ToN(5);        // 1 2 3 4 5
console.log(collect1ToN(5)); // [1, 2, 3, 4, 5]
```

## 🎨 Step-by-Step Walkthrough

### Fibonacci(5) Call Stack:

```
fib(5)
├── fib(4)
│   ├── fib(3)
│   │   ├── fib(2)
│   │   │   ├── fib(1) = 1
│   │   │   └── fib(0) = 0
│   │   │   Returns: 1
│   │   └── fib(1) = 1
│   │   Returns: 2
│   └── fib(2)
│       ├── fib(1) = 1
│       └── fib(0) = 0
│       Returns: 1
│   Returns: 3
└── fib(3)
    ├── fib(2)
    │   ├── fib(1) = 1
    │   └── fib(0) = 0
    │   Returns: 1
    └── fib(1) = 1
    Returns: 2
Returns: 5
```

### Print 1 to 5 Call Stack:

```
print1ToN(5)
└── print1ToN(4)
    └── print1ToN(3)
        └── print1ToN(2)
            └── print1ToN(1)
                └── print1ToN(0) // Base case, returns
                Prints: 1
            Prints: 2
        Prints: 3
    Prints: 4
Prints: 5
```

## 📊 Visual Understanding

```
Fibonacci with Memoization:

First call: fib(5)
Calculates: fib(4) and fib(3)

When calculating fib(4):
- Needs fib(3) and fib(2)
- fib(3) calculated and stored

When calculating fib(5)'s fib(3):
- Already in memo! Return immediately

Tree of calls without memo:    With memo:
        fib(5)                     fib(5)
       /      \                   /      \
    fib(4)    fib(3)           fib(4)    fib(3)✓
    /    \    /    \           /    \      ↑
  fib(3) fib(2) ...          fib(3) fib(2) (cached)
  /   \                      /   \
fib(2) fib(1)              fib(2) fib(1)

Calls: 15 (naive)          Calls: 9 (memoized)
```

## ⏱️ Complexity Analysis

| Method | Time | Space | Notes |
|--------|------|-------|-------|
| Fibonacci Naive | O(2^n) | O(n) | Exponential time, stack depth |
| Fibonacci Memo | O(n) | O(n) | Linear time, memo storage |
| Fibonacci Iterative | O(n) | O(1) | Best space complexity |
| Fibonacci Matrix | O(log n) | O(log n) | Fastest for large n |
| Print 1 to N | O(n) | O(n) | Linear time, stack depth |

## 🚫 Common Mistakes

1. **Missing base cases**
   ```javascript
   // Wrong: No base case
   function fib(n) {
       return fib(n-1) + fib(n-2); // Infinite recursion!
   }
   
   // Right: Include base cases
   function fib(n) {
       if (n <= 1) return n;
       return fib(n-1) + fib(n-2);
   }
   ```

2. **Memoization object reference**
   ```javascript
   // Wrong: New memo each call
   function fib(n) {
       const memo = {};  // Lost between calls!
   
   // Right: Pass memo through calls
   function fib(n, memo = {}) {
   ```

3. **Stack overflow for large n**
   ```javascript
   // Wrong: Deep recursion
   fibonacciNaive(50); // Stack overflow!
   
   // Right: Use iteration or memoization
   fibonacciIterative(50); // Works fine
   ```

## 🔄 Variations

```javascript
// Tribonacci (sum of three previous)
function tribonacci(n) {
    if (n === 0) return 0;
    if (n === 1 || n === 2) return 1;
    
    const memo = {0: 0, 1: 1, 2: 1};
    
    function trib(n) {
        if (n in memo) return memo[n];
        memo[n] = trib(n-1) + trib(n-2) + trib(n-3);
        return memo[n];
    }
    
    return trib(n);
}

// Print even/odd numbers
function printEven1ToN(n) {
    if (n <= 0) return;
    
    printEven1ToN(n - 1);
    if (n % 2 === 0) {
        console.log(n);
    }
}

// Sum using recursion
function sum1ToN(n) {
    if (n <= 0) return 0;
    return n + sum1ToN(n - 1);
}

// Factorial (similar pattern)
function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

## 🔗 Related Problems

- **Climbing Stairs** - Fibonacci in disguise
- **House Robber** - DP with Fibonacci-like recurrence
- **Pascal's Triangle** - Recursive number patterns
- **Power of Number** - Recursive multiplication

## 💡 Key Takeaways

1. **Base Cases are Critical**: Prevent infinite recursion
2. **Memoization**: Transform exponential to linear time
3. **Call Stack Visualization**: Understand execution order
4. **Iterative Alternative**: Often more space-efficient
5. **Pattern Recognition**: Many problems reduce to Fibonacci-like patterns

These fundamental recursion problems teach the core concepts needed for more complex recursive algorithms!
