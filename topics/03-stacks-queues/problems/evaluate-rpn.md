# Evaluate Reverse Polish Notation (RPN)

## 🎯 What Are We Trying To Solve?

Imagine you're building a calculator that reads expressions written in a special way where operators come AFTER the numbers they operate on. Instead of writing "3 + 4", you write "3 4 +". This is how old HP calculators worked, and it's actually easier for computers to evaluate!

## 📝 The Problem

Evaluate the value of an arithmetic expression in Reverse Polish Notation (postfix notation). Valid operators are `+`, `-`, `*`, and `/`. Each operand may be an integer or another expression.

```
Examples:
["2","1","+","3","*"] → ((2 + 1) * 3) = 9
["4","13","5","/","+"] → (4 + (13 / 5)) = 6
["10","6","9","3","+","-11","*","/","*","17","+","5","+"] → 22
```

## 🧠 How to Think About This

RPN eliminates the need for parentheses! When you see an operator, it operates on the two most recent values. Use a stack to track values - when you see an operator, pop two values, compute, and push the result back.

### Visual Process
```
Input: ["2", "1", "+", "3", "*"]

Step 1: "2" → Stack: [2]
Step 2: "1" → Stack: [2, 1]
Step 3: "+" → Pop 1, 2 → Calculate 2+1=3 → Stack: [3]
Step 4: "3" → Stack: [3, 3]
Step 5: "*" → Pop 3, 3 → Calculate 3*3=9 → Stack: [9]

Result: 9
```

## 💻 The Code

```javascript
/**
 * Evaluate expression in Reverse Polish Notation
 * Time: O(n) - Process each token once
 * Space: O(n) - Stack can hold up to n/2 operands
 */
function evalRPN(tokens) {
    const stack = [];
    
    // Define operators and their functions
    const operators = {
        '+': (a, b) => a + b,
        '-': (a, b) => a - b,
        '*': (a, b) => a * b,
        '/': (a, b) => Math.trunc(a / b)  // Truncate toward zero
    };
    
    for (const token of tokens) {
        if (token in operators) {
            // It's an operator - pop two operands
            // IMPORTANT: Order matters for - and /
            const b = stack.pop();  // Second operand (right)
            const a = stack.pop();  // First operand (left)
            
            // Apply operator and push result
            const result = operators[token](a, b);
            stack.push(result);
        } else {
            // It's a number - push to stack
            stack.push(parseInt(token));
        }
    }
    
    // Final result is the only element left
    return stack[0];
}

// Test cases
console.log(evalRPN(["2","1","+","3","*"])); // 9
console.log(evalRPN(["4","13","5","/","+"])); // 6
console.log(evalRPN(["10","6","9","3","+","-11","*","/","*","17","+","5","+"])); // 22
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `["4","13","5","/","+"]`:

```
Infix equivalent: 4 + (13 / 5)

Token "4":
- It's a number
- Push to stack
- Stack: [4]

Token "13":
- It's a number
- Push to stack
- Stack: [4, 13]

Token "5":
- It's a number
- Push to stack
- Stack: [4, 13, 5]

Token "/":
- It's an operator
- Pop b = 5, a = 13
- Calculate: 13 / 5 = 2 (truncated)
- Push result
- Stack: [4, 2]

Token "+":
- It's an operator
- Pop b = 2, a = 4
- Calculate: 4 + 2 = 6
- Push result
- Stack: [6]

Result: 6 ✓
```

## 📊 Why RPN is Efficient

### No Parentheses Needed!
```
Infix:    ((15 ÷ (7 − (1 + 1))) × 3) − (2 + (1 + 1))
RPN:      15 7 1 1 + − ÷ 3 × 2 1 1 + + −

The RPN version:
- No parentheses
- No precedence rules
- Linear evaluation
```

### Comparison of Notations
```
Infix (normal):     3 + 4 * 2
Need precedence:    3 + (4 * 2) = 11

Prefix (Polish):    + 3 * 4 2
Operator first:     +(3, *(4, 2)) = 11

Postfix (RPN):      3 4 2 * +
Operator last:      3, (4*2), + = 11
```

## 🚫 Common Mistakes

1. **Wrong Order for Non-Commutative Operations**
   ```javascript
   // Wrong: Popping in wrong order
   const a = stack.pop(); // This is actually the second operand!
   const b = stack.pop(); // This is the first operand!
   return a - b; // Wrong: should be b - a
   
   // Right: Pop in correct order
   const b = stack.pop(); // Second (right) operand
   const a = stack.pop(); // First (left) operand
   return a - b; // Correct order
   ```

2. **Integer Division Rounding**
   ```javascript
   // Wrong: JavaScript's default division
   '/': (a, b) => a / b  // Returns float: 13/5 = 2.6
   
   // Right: Truncate toward zero
   '/': (a, b) => Math.trunc(a / b)  // Returns 2
   ```

3. **Not Handling Negative Numbers**
   ```javascript
   // Wrong: Will treat "-11" as operator
   if (['+','-','*','/'].includes(token))
   
   // Right: Check if it's in operators object
   if (token in operators)
   // Or: Try parsing first
   const num = parseInt(token);
   if (!isNaN(num))
   ```

## 🔄 Alternative Approaches

### Recursive Evaluation
```javascript
function evalRPNRecursive(tokens) {
    let index = tokens.length - 1;
    
    function evaluate() {
        const token = tokens[index--];
        
        if (['+','-','*','/'].includes(token)) {
            // Operator: evaluate right then left (reverse order!)
            const right = evaluate();
            const left = evaluate();
            
            switch(token) {
                case '+': return left + right;
                case '-': return left - right;
                case '*': return left * right;
                case '/': return Math.trunc(left / right);
            }
        } else {
            // Operand
            return parseInt(token);
        }
    }
    
    return evaluate();
}
```

### Convert to Infix First (Educational)
```javascript
function rpnToInfix(tokens) {
    const stack = [];
    
    for (const token of tokens) {
        if (['+','-','*','/'].includes(token)) {
            const b = stack.pop();
            const a = stack.pop();
            stack.push(`(${a}${token}${b})`);
        } else {
            stack.push(token);
        }
    }
    
    return stack[0];
}

// ["2","1","+","3","*"] → "((2+1)*3)"
```

## 🔗 Related Problems

- **Basic Calculator I, II, III** - Evaluate infix expressions
- **Decode String** - Stack-based string manipulation
- **Parse Lisp Expression** - Expression evaluation
- **Number of Atoms** - Parse chemical formulas

## 💡 Key Takeaways

1. **Stack is Natural for RPN**: Push operands, pop for operators
2. **Order Matters**: For `-` and `/`, pop order is crucial
3. **No Precedence Needed**: RPN eliminates ambiguity
4. **Linear Evaluation**: Single pass, no backtracking

RPN is used in stack-based languages, some calculators, and internally by compilers!
