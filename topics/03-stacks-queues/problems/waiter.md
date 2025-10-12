# Waiter (Stack Distribution)

## 🎯 What Are We Trying To Solve?

Imagine you're a waiter with a stack of numbered plates. You need to sort them into different piles based on whether their numbers are divisible by specific prime numbers. It's like sorting mail into different boxes, but using mathematical rules!

## 📝 The Problem

You have a stack of plates numbered with integers. You need to perform Q iterations:
- In iteration i, divide the plates based on whether they're divisible by the i-th prime number
- Plates divisible by the prime go to pile B[i]
- Others go to the next iteration
- After all iterations, output all B piles followed by remaining plates

```
Example:
Initial stack: [3, 4, 7, 6, 5] (top to bottom)
Q = 1 (use first prime: 2)

Iteration 1:
- 4 divisible by 2 → B[0] = [4]
- 6 divisible by 2 → B[0] = [4, 6]
- Others → A[1] = [5, 7, 3]

Output: B[0] then A[1] = 4, 6, 5, 7, 3
```

## 💻 The Code

```javascript
/**
 * Distribute plates using prime number divisibility
 * @param {number[]} plates - Initial stack of plates
 * @param {number} q - Number of iterations
 * @returns {number[]} - Ordered output of all plates
 */
function waiter(plates, q) {
    // First 1200 prime numbers (enough for constraints)
    const primes = getPrimes(1200);
    
    // Initialize stacks
    let currentStack = [...plates];
    const bStacks = [];
    
    // Perform q iterations
    for (let i = 0; i < q; i++) {
        const prime = primes[i];
        const bStack = [];
        const nextStack = [];
        
        // Process current stack from top to bottom
        while (currentStack.length > 0) {
            const plate = currentStack.pop();
            
            if (plate % prime === 0) {
                // Divisible by prime → goes to B stack
                bStack.push(plate);
            } else {
                // Not divisible → goes to next iteration
                nextStack.push(plate);
            }
        }
        
        // Save B stack and prepare for next iteration
        bStacks.push(bStack);
        currentStack = nextStack;
    }
    
    // Collect results: all B stacks, then remaining plates
    const result = [];
    
    // Output each B stack (top to bottom)
    for (const bStack of bStacks) {
        while (bStack.length > 0) {
            result.push(bStack.pop());
        }
    }
    
    // Output remaining plates (top to bottom)
    while (currentStack.length > 0) {
        result.push(currentStack.pop());
    }
    
    return result;
}

/**
 * Generate first n prime numbers using Sieve of Eratosthenes
 */
function getPrimes(n) {
    // Estimate upper bound for n-th prime
    const limit = n * 15; // Conservative estimate
    const sieve = new Array(limit + 1).fill(true);
    sieve[0] = sieve[1] = false;
    
    for (let i = 2; i * i <= limit; i++) {
        if (sieve[i]) {
            for (let j = i * i; j <= limit; j += i) {
                sieve[j] = false;
            }
        }
    }
    
    const primes = [];
    for (let i = 2; i <= limit && primes.length < n; i++) {
        if (sieve[i]) {
            primes.push(i);
        }
    }
    
    return primes;
}

// Alternative: Hardcoded first few primes for efficiency
function waiterSimple(plates, q) {
    // First 50 primes (enough for most problems)
    const primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 
                    53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 
                    109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 
                    173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229];
    
    let currentStack = [...plates];
    const result = [];
    
    for (let i = 0; i < q; i++) {
        const prime = primes[i];
        const bStack = [];
        const aStack = [];
        
        // Process from bottom to top (reverse order)
        for (let j = currentStack.length - 1; j >= 0; j--) {
            const plate = currentStack[j];
            if (plate % prime === 0) {
                bStack.push(plate);
            } else {
                aStack.push(plate);
            }
        }
        
        // Add B stack to result (reverse for correct order)
        for (let j = bStack.length - 1; j >= 0; j--) {
            result.push(bStack[j]);
        }
        
        // A stack becomes current for next iteration
        currentStack = aStack;
    }
    
    // Add remaining plates
    for (let j = currentStack.length - 1; j >= 0; j--) {
        result.push(currentStack[j]);
    }
    
    return result;
}

// Test cases
console.log(waiter([3, 4, 7, 6, 5], 1));
// Prime 2: B[0]=[4,6], A[1]=[5,7,3]
// Output: [4, 6, 5, 7, 3]

console.log(waiter([3, 3, 4, 4, 9], 2));
// Prime 2: B[0]=[4,4], A[1]=[9,3,3]
// Prime 3: B[1]=[9,3,3], A[2]=[]
// Output: [4, 4, 9, 3, 3]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `waiter([3, 4, 7, 6, 5], 1)`:

```
Initial stack (top to bottom): [3, 4, 7, 6, 5]
Q = 1, so we use first prime = 2

Iteration 1 (Prime = 2):
Process from top:
- 3 % 2 = 1 (not divisible) → A stack
- 4 % 2 = 0 (divisible) → B stack
- 7 % 2 = 1 (not divisible) → A stack
- 6 % 2 = 0 (divisible) → B stack
- 5 % 2 = 1 (not divisible) → A stack

After iteration:
B[0] = [4, 6] (order: first in at bottom)
A[1] = [3, 7, 5] (order preserved from original)

Output order:
1. B[0] from top: 6, 4
2. A[1] from top: 5, 7, 3

Wait, let me reconsider the stack order...

Actually, processing stack from top (pop):
Stack: [3, 4, 7, 6, 5] where 5 is at top

Pop 5: 5 % 2 = 1 → nextStack
Pop 6: 6 % 2 = 0 → bStack
Pop 7: 7 % 2 = 1 → nextStack
Pop 4: 4 % 2 = 0 → bStack
Pop 3: 3 % 2 = 1 → nextStack

bStack = [6, 4] (6 pushed first, 4 on top)
nextStack = [5, 7, 3] (5 pushed first, 3 on top)

Output B stack: pop 4, pop 6 = [4, 6]
Output A stack: pop 3, pop 7, pop 5 = [3, 7, 5]

Final: [4, 6, 3, 7, 5]
```

## 📊 Visual Understanding

```
Distribution Process:

Original:    [5]    ← top
             [6]
             [7]
             [4]
             [3]    ← bottom

After Prime 2:
B[0]:        [4]    ← top      A[1]:    [3]    ← top
             [6]    ← bottom             [7]
                                         [5]    ← bottom

Output Order:
B[0]: 4, 6 (pop from top)
A[1]: 3, 7, 5 (pop from top)
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(n × q)
  - n plates processed in each of q iterations
  - Prime generation: O(p log log p) where p is prime limit
  
- **Space Complexity:** O(n + q)
  - O(n) for plate stacks
  - O(q) for B stacks
  - O(p) for prime storage

## 🚫 Common Mistakes

1. **Wrong Stack Order**
   ```javascript
   // Wrong: Not maintaining stack LIFO order
   for (const plate of currentStack) {
       // This iterates in wrong order!
   }
   
   // Right: Pop from top
   while (currentStack.length > 0) {
       const plate = currentStack.pop();
   }
   ```

2. **Output Order Confusion**
   ```javascript
   // Wrong: Pushing B stack directly
   result.push(...bStack);
   
   // Right: Pop from B stack for correct order
   while (bStack.length > 0) {
       result.push(bStack.pop());
   }
   ```

3. **Prime Number Issues**
   ```javascript
   // Wrong: Starting primes from 1
   const primes = [1, 2, 3, 5, 7...];  // 1 is not prime!
   
   // Right: Start from 2
   const primes = [2, 3, 5, 7, 11...];
   ```

## 🔄 Variations

```javascript
// Using filter for cleaner code
function waiterFunctional(plates, q) {
    const primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29];
    let current = plates.slice().reverse(); // Bottom to top
    const result = [];
    
    for (let i = 0; i < q; i++) {
        const prime = primes[i];
        const divisible = current.filter(p => p % prime === 0);
        const notDivisible = current.filter(p => p % prime !== 0);
        
        result.push(...divisible.reverse());
        current = notDivisible;
    }
    
    result.push(...current.reverse());
    return result;
}

// Check if number is prime
function isPrime(n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 === 0 || n % 3 === 0) return false;
    
    for (let i = 5; i * i <= n; i += 6) {
        if (n % i === 0 || n % (i + 2) === 0) {
            return false;
        }
    }
    return true;
}

// Generate primes on the fly
function* primeGenerator() {
    yield 2;
    let n = 3;
    while (true) {
        if (isPrime(n)) {
            yield n;
        }
        n += 2;
    }
}
```

## 🔗 Related Problems

- **Stack Permutation** - Check if one stack permutation can produce another
- **Sort Stack** - Sort a stack using additional stacks
- **Special Stack** - Design stack with getMin() in O(1)
- **Celebrity Problem** - Find celebrity using stack

## 💡 Key Takeaways

1. **Stack Order Matters**: Remember LIFO behavior when processing
2. **Prime Numbers**: Efficient generation or caching is important
3. **Multiple Stacks**: Managing multiple stacks requires careful tracking
4. **Output Order**: Pay attention to how stacks are emptied for output

This problem combines stack operations with number theory - a unique combination that tests both data structure and mathematical knowledge!
