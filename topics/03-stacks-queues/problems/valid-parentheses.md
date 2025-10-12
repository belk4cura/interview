# Valid Parentheses

## 🎯 What Are We Trying To Solve?

Imagine you're checking if all the brackets in your code are properly matched - every opening bracket has a corresponding closing bracket in the right order. It's like making sure every open door is eventually closed, and you can't close a door that was never opened!

## 📝 The Problem

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

A string is valid if:
1. Open brackets must be closed by the same type of brackets
2. Open brackets must be closed in the correct order
3. Every close bracket has a corresponding open bracket

```
Examples:
"()" → true
"()[]{}" → true
"(]" → false (wrong type)
"([)]" → false (wrong order)
"{[()]}" → true (properly nested)
```

## 🧠 How to Think About This

Think of a stack of plates - you can only add or remove from the top. When we see an opening bracket, we "remember" it by putting it on the stack. When we see a closing bracket, we check if it matches what's on top of the stack.

### Visual Process
```
Input: "{[()]}"

Step 1: { → Stack: [{]
Step 2: [ → Stack: [{, []
Step 3: ( → Stack: [{, [, (]
Step 4: ) → Match with (, pop → Stack: [{, []
Step 5: ] → Match with [, pop → Stack: [{]
Step 6: } → Match with {, pop → Stack: []

Stack empty = Valid! ✓
```

## 💻 The Code

```javascript
/**
 * Check if parentheses/brackets are valid
 * Time: O(n) - Process each character once
 * Space: O(n) - Stack can hold up to n/2 opening brackets
 */
function isValid(s) {
    // Stack to keep track of opening brackets
    const stack = [];
    
    // Map to match opening with closing brackets
    const pairs = {
        '(': ')',
        '[': ']',
        '{': '}'
    };
    
    // Process each character
    for (const char of s) {
        if (char in pairs) {
            // It's an opening bracket - push to stack
            stack.push(char);
        } else {
            // It's a closing bracket
            
            // No opening bracket to match
            if (stack.length === 0) {
                return false;
            }
            
            // Pop the last opening bracket
            const lastOpening = stack.pop();
            
            // Check if it matches current closing bracket
            if (pairs[lastOpening] !== char) {
                return false;
            }
        }
    }
    
    // Valid if all brackets were matched (stack empty)
    return stack.length === 0;
}

// Test cases
console.log(isValid("()"));        // true
console.log(isValid("()[]{}"));    // true
console.log(isValid("(]"));        // false
console.log(isValid("([)]"));      // false
console.log(isValid("{[()]}"));    // true
console.log(isValid("(("));        // false (unclosed)
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `"([)]"` (which should be false):

```
Input: "([)]"
Stack: []

Step 1: Process '('
- It's an opening bracket
- Push to stack
- Stack: ['(']

Step 2: Process '['
- It's an opening bracket
- Push to stack
- Stack: ['(', '[']

Step 3: Process ')'
- It's a closing bracket
- Pop from stack: '[' 
- Check: pairs['['] = ']', but we have ')'
- Mismatch! Return false ✗

The problem: We're trying to close '(' but '[' is in the way!
```

Now let's trace a valid example `"([])""`:

```
Input: "([])"
Stack: []

Step 1: '(' → Stack: ['(']
Step 2: '[' → Stack: ['(', '[']
Step 3: ']' → Pop '[', matches! → Stack: ['(']
Step 4: ')' → Pop '(', matches! → Stack: []

Stack empty = Valid! ✓
```

## 📊 Why the Stack Works

The stack enforces the LIFO (Last In, First Out) order, which is exactly what we need for matching brackets:

```
Nested Structure:    { [ ( ) ] }
                     ↓ ↓ ↓ ↑ ↑ ↑
Stack Growth:        1 2 3 3 2 1

The innermost bracket opened last must close first!
```

## 🚫 Common Mistakes

1. **Forgetting to Check Empty Stack**
   ```javascript
   // Wrong: Will crash on "]" (no opening bracket)
   const lastOpening = stack.pop(); // pop() on empty array returns undefined
   
   // Right: Check first
   if (stack.length === 0) return false;
   const lastOpening = stack.pop();
   ```

2. **Not Checking if Stack is Empty at End**
   ```javascript
   // Wrong: "(((" would incorrectly return true
   return true;
   
   // Right: Check if all brackets were closed
   return stack.length === 0;
   ```

3. **Using Wrong Data Structure**
   ```javascript
   // Wrong: Queue (FIFO) doesn't work for nested structures
   const queue = [];
   queue.push(bracket);     // Add to end
   queue.shift();           // Remove from front - WRONG ORDER!
   
   // Right: Stack (LIFO)
   const stack = [];
   stack.push(bracket);     // Add to end
   stack.pop();            // Remove from end - CORRECT!
   ```

## 🔄 Alternative Approaches

### Approach 2: Counter Method (Only for Single Type)
```javascript
// Works ONLY for single bracket type like "()"
function isValidSingleType(s) {
    let counter = 0;
    
    for (const char of s) {
        if (char === '(') {
            counter++;
        } else {
            counter--;
            if (counter < 0) return false; // More closing than opening
        }
    }
    
    return counter === 0;
}
```

### Approach 3: Replace Method (Inefficient but Intuitive)
```javascript
// Keep removing valid pairs until none left
function isValidReplace(s) {
    while (s.includes('()') || s.includes('[]') || s.includes('{}')) {
        s = s.replace('()', '');
        s = s.replace('[]', '');
        s = s.replace('{}', '');
    }
    return s === '';
}
// Time: O(n²) - Much slower than stack approach!
```

## 🔗 Related Problems

- **Generate Parentheses** - Generate all valid combinations
- **Longest Valid Parentheses** - Find longest valid substring
- **Remove Invalid Parentheses** - Minimum removals to make valid
- **Score of Parentheses** - Calculate score based on nesting

## 💡 Key Takeaways

1. **Stack is Perfect for Matching**: LIFO order matches nesting structure
2. **Process Character by Character**: Single pass through the string
3. **Early Exit on Mismatch**: Return false as soon as invalid
4. **Check Final State**: Stack must be empty for validity

This is a classic stack problem that appears in many forms - HTML tag matching, compiler syntax checking, and more!
