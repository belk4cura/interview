# Designer PDF Viewer

## 🎯 What Are We Trying To Solve?

Imagine you're highlighting text in a PDF viewer. Each letter has a different height (some letters like 'l' are tall, others like 'a' are short). When you highlight a word, the highlight box must be tall enough for the tallest letter and wide enough for all letters. We need to calculate the area of this highlight box!

## 📝 The Problem

You are given:
1. An array of 26 integers, where each represents the height of letters a-z (in order)
2. A word to highlight

Calculate the area of the highlight rectangle, which is:
- Height: The height of the tallest letter in the word
- Width: The number of letters in the word
- Area: height × width

## 📊 Examples

**Example 1:**
```
Heights: [1,3,1,3,1,4,1,3,2,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5]
         (a b c d e f g h i j k l m n o p q r s t u v w x y z)
Word: "abc"

Tallest letter: 'b' with height 3
Word length: 3
Area: 3 × 3 = 9
```

**Example 2:**
```
Heights: [1,3,1,3,1,4,1,3,2,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,7]
Word: "zaba"

Letter heights: z=7, a=1, b=3, a=1
Tallest letter: 'z' with height 7
Word length: 4
Area: 7 × 4 = 28
```

## 💻 The Code

```javascript
/**
 * Calculate the area of highlighted text in PDF viewer
 * @param {number[]} heights - Array of 26 heights for letters a-z
 * @param {string} word - The word to highlight
 * @returns {number} - Area of the highlight rectangle
 */
function designerPdfViewer(heights, word) {
    let maxHeight = 0;
    
    // Check each letter in the word
    for (let i = 0; i < word.length; i++) {
        // Convert letter to index (0-25)
        // 'a'.charCodeAt(0) = 97, 'b'.charCodeAt(0) = 98, etc.
        // So 'b' - 'a' = 98 - 97 = 1 (index for 'b')
        const letterIndex = word.charCodeAt(i) - 'a'.charCodeAt(0);
        
        // Get height for this letter
        const letterHeight = heights[letterIndex];
        
        // Update max height if this letter is taller
        if (letterHeight > maxHeight) {
            maxHeight = letterHeight;
        }
    }
    
    // Area = max height × word length
    return maxHeight * word.length;
}

// Alternative: Using Math.max and map
function designerPdfViewerFunctional(heights, word) {
    // Convert each letter to its height, then find max
    const letterHeights = word.split('').map(letter => {
        const index = letter.charCodeAt(0) - 'a'.charCodeAt(0);
        return heights[index];
    });
    
    const maxHeight = Math.max(...letterHeights);
    return maxHeight * word.length;
}

// Alternative: More concise version
function designerPdfViewerConcise(heights, word) {
    const maxHeight = Math.max(
        ...word.split('').map(ch => heights[ch.charCodeAt(0) - 97])
    );
    return maxHeight * word.length;
}

// Test cases
const heights1 = [1,3,1,3,1,4,1,3,2,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5];
console.log(designerPdfViewer(heights1, "abc"));  // 9

const heights2 = [1,3,1,3,1,4,1,3,2,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,5,7];
console.log(designerPdfViewer(heights2, "zaba")); // 28
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `designerPdfViewer([1,3,1,3,1,4,...], "abc")`:

```
Initial: maxHeight = 0, word = "abc"
Heights array: a=1, b=3, c=1, d=3, ...

Step 1: Process 'a'
- letterIndex = 'a' - 'a' = 0
- letterHeight = heights[0] = 1
- maxHeight = max(0, 1) = 1

Step 2: Process 'b'
- letterIndex = 'b' - 'a' = 1
- letterHeight = heights[1] = 3
- maxHeight = max(1, 3) = 3

Step 3: Process 'c'
- letterIndex = 'c' - 'a' = 2
- letterHeight = heights[2] = 1
- maxHeight = max(3, 1) = 3

Calculate area: 3 × 3 = 9
```

## 🔤 Understanding Character Arithmetic

```javascript
// How character to index conversion works:
console.log('a'.charCodeAt(0));  // 97 (ASCII code)
console.log('b'.charCodeAt(0));  // 98
console.log('z'.charCodeAt(0));  // 122

// Converting to 0-based index:
console.log('a'.charCodeAt(0) - 'a'.charCodeAt(0));  // 0
console.log('b'.charCodeAt(0) - 'a'.charCodeAt(0));  // 1
console.log('z'.charCodeAt(0) - 'a'.charCodeAt(0));  // 25

// Shorthand (since 'a' = 97):
console.log('b'.charCodeAt(0) - 97);  // 1
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(n) where n is the length of the word
  - We check each letter once
  
- **Space Complexity:** O(1)
  - We only use a single variable for max height
  - The functional approach uses O(n) for the intermediate array

## 🚫 Common Mistakes

1. **Off-by-one errors in character conversion**
   ```javascript
   // Wrong: Forgetting to subtract 'a'
   const index = word.charCodeAt(i);  // This gives 97+ not 0-25!
   
   // Right: Convert to 0-based index
   const index = word.charCodeAt(i) - 'a'.charCodeAt(0);
   ```

2. **Using wrong ASCII value**
   ```javascript
   // Wrong: Using capital 'A' (ASCII 65)
   const index = word.charCodeAt(i) - 'A'.charCodeAt(0);
   
   // Right: Use lowercase 'a' (ASCII 97)
   const index = word.charCodeAt(i) - 'a'.charCodeAt(0);
   ```

3. **Calculating area incorrectly**
   ```javascript
   // Wrong: Adding instead of multiplying
   return maxHeight + word.length;
   
   // Right: Area = height × width
   return maxHeight * word.length;
   ```

## 🔄 Variations

```javascript
// Variation 1: Return which letter is tallest
function tallestLetter(heights, word) {
    let maxHeight = 0;
    let tallestChar = '';
    
    for (const char of word) {
        const height = heights[char.charCodeAt(0) - 97];
        if (height > maxHeight) {
            maxHeight = height;
            tallestChar = char;
        }
    }
    
    return { char: tallestChar, height: maxHeight };
}

// Variation 2: Handle uppercase letters too
function designerPdfViewerMixedCase(heights, word) {
    const lowerWord = word.toLowerCase();
    let maxHeight = 0;
    
    for (const char of lowerWord) {
        if (char >= 'a' && char <= 'z') {
            const height = heights[char.charCodeAt(0) - 97];
            maxHeight = Math.max(maxHeight, height);
        }
    }
    
    return maxHeight * lowerWord.length;
}

// Variation 3: Calculate total ink used (sum of all heights)
function totalInkUsed(heights, word) {
    return word.split('').reduce((sum, char) => {
        return sum + heights[char.charCodeAt(0) - 97];
    }, 0);
}
```

## 🔗 Related Problems

- **Maximum Element in Array** - Finding max value
- **Character Frequency** - Counting character occurrences
- **String Manipulation** - Working with character codes
- **Rectangle Area** - Basic geometry calculations

## 💡 Key Takeaways

1. **Character Arithmetic**: Converting letters to array indices is a common pattern
2. **Finding Maximum**: Track the max value while iterating
3. **Simple Math**: Sometimes the solution is just basic arithmetic
4. **ASCII Knowledge**: Understanding character codes helps in many problems

This problem combines array indexing, string manipulation, and basic math - all fundamental skills for coding interviews!
