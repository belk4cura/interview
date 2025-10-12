# Binary Search Tree Insertion

## 🎯 What Are We Trying To Solve?

Inserting a node into a Binary Search Tree (BST) is like finding the right spot for a book in a sorted bookshelf - you navigate left or right based on comparisons until you find the perfect empty spot. The BST property must be maintained: left children are smaller, right children are larger.

## 📝 The Problem

Given a BST and a value to insert, add a new node with that value while maintaining the BST property. The insertion should follow these rules:
- If the tree is empty, the new node becomes the root
- If the value is less than current node, go left
- If the value is greater than current node, go right
- Insert at the first empty spot found

```
Example:
BST:     4           Insert: 6
        / \
       2   7         Result:     4
      / \                       / \
     1   3                     2   7
                              / \ /
                             1  3 6
```

## 💻 The Code

```javascript
// Definition for a binary tree node
class TreeNode {
    constructor(val = 0, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

/**
 * Insert a value into BST (recursive)
 * @param {TreeNode} root - Root of BST
 * @param {number} val - Value to insert
 * @returns {TreeNode} - Root of modified BST
 */
function insertIntoBST(root, val) {
    // Base case: empty tree or found insertion point
    if (root === null) {
        return new TreeNode(val);
    }
    
    // Navigate to correct subtree
    if (val < root.val) {
        // Go left
        root.left = insertIntoBST(root.left, val);
    } else if (val > root.val) {
        // Go right
        root.right = insertIntoBST(root.right, val);
    }
    // If val === root.val, we typically don't insert duplicates
    
    return root;
}

/**
 * Insert iteratively (often more space efficient)
 * @param {TreeNode} root - Root of BST
 * @param {number} val - Value to insert
 * @returns {TreeNode} - Root of modified BST
 */
function insertIntoBSTIterative(root, val) {
    const newNode = new TreeNode(val);
    
    // Handle empty tree
    if (root === null) {
        return newNode;
    }
    
    let current = root;
    while (true) {
        if (val < current.val) {
            // Go left
            if (current.left === null) {
                current.left = newNode;
                break;
            }
            current = current.left;
        } else if (val > current.val) {
            // Go right
            if (current.right === null) {
                current.right = newNode;
                break;
            }
            current = current.right;
        } else {
            // Duplicate value, don't insert
            break;
        }
    }
    
    return root;
}

/**
 * Insert with parent tracking (useful for deletion)
 */
function insertWithParent(root, val) {
    const newNode = new TreeNode(val);
    
    if (root === null) {
        return newNode;
    }
    
    let parent = null;
    let current = root;
    
    // Find insertion point
    while (current !== null) {
        parent = current;
        if (val < current.val) {
            current = current.left;
        } else if (val > current.val) {
            current = current.right;
        } else {
            // Duplicate found
            return root;
        }
    }
    
    // Insert new node
    if (val < parent.val) {
        parent.left = newNode;
    } else {
        parent.right = newNode;
    }
    
    return root;
}

/**
 * Insert multiple values
 */
function insertMultiple(root, values) {
    for (const val of values) {
        root = insertIntoBST(root, val);
    }
    return root;
}

/**
 * Build BST from array
 */
function buildBST(values) {
    if (values.length === 0) return null;
    
    let root = null;
    for (const val of values) {
        root = insertIntoBST(root, val);
    }
    return root;
}

// Helper functions for testing
function inorderTraversal(root) {
    if (root === null) return [];
    return [
        ...inorderTraversal(root.left),
        root.val,
        ...inorderTraversal(root.right)
    ];
}

function levelOrder(root) {
    if (!root) return [];
    
    const result = [];
    const queue = [root];
    
    while (queue.length > 0) {
        const level = [];
        const levelSize = queue.length;
        
        for (let i = 0; i < levelSize; i++) {
            const node = queue.shift();
            level.push(node.val);
            
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        
        result.push(level);
    }
    
    return result;
}

// Test cases
const root = buildBST([4, 2, 7, 1, 3]);
insertIntoBST(root, 6);
console.log(inorderTraversal(root)); // [1, 2, 3, 4, 6, 7]

const root2 = buildBST([40, 20, 60, 10, 30, 50, 70]);
insertIntoBST(root2, 25);
console.log(levelOrder(root2)); 
// [[40], [20, 60], [10, 30, 50, 70], [25]]
```

## 🎨 Step-by-Step Walkthrough

Let's trace inserting 6 into this BST:

```
Initial BST:
       4
      / \
     2   7
    / \
   1   3

Insert 6:

Step 1: Start at root (4)
- 6 > 4, go right to 7

Step 2: At node 7
- 6 < 7, go left
- Left is null, insert here!

Step 3: Create new node
- 7.left = new TreeNode(6)

Final BST:
       4
      / \
     2   7
    / \ /
   1  3 6
```

## 📊 Visual Understanding

```
Insertion Path for value 5:

       8
      / \
     3   10        5 < 8, go left
    / \    \
   1   6    14     5 > 3, go right
      / \   /
     4   7 13      5 < 6, go left
    /
   ?               5 > 4, go right
                   Right is null, insert!
       8
      / \
     3   10
    / \    \
   1   6    14
      / \   /
     4   7 13
      \
       5 ← New node
```

## ⏱️ Complexity Analysis

- **Time Complexity:**
  - Average: O(log n) - Balanced tree
  - Worst: O(n) - Skewed tree (linked list)
  
- **Space Complexity:**
  - Recursive: O(h) where h is height (call stack)
  - Iterative: O(1) - No recursion stack

## 🚫 Common Mistakes

1. **Not returning the root**
   ```javascript
   // Wrong: Doesn't return root for empty tree
   function insert(root, val) {
       if (root === null) {
           new TreeNode(val); // Lost reference!
       }
   }
   
   // Right: Return the new node
   if (root === null) {
       return new TreeNode(val);
   }
   ```

2. **Losing tree connections**
   ```javascript
   // Wrong: Doesn't connect new subtree
   insertIntoBST(root.left, val);
   
   // Right: Reconnect the modified subtree
   root.left = insertIntoBST(root.left, val);
   ```

3. **Inserting duplicates incorrectly**
   ```javascript
   // Consider: How to handle duplicates?
   // Option 1: Ignore (most common)
   if (val === root.val) return root;
   
   // Option 2: Allow (treat as greater)
   if (val >= root.val) root.right = insert(root.right, val);
   
   // Option 3: Count frequency
   if (val === root.val) root.count++;
   ```

## 🔄 Variations

```javascript
// Insert and return the new node
function insertAndReturnNode(root, val) {
    if (root === null) {
        return new TreeNode(val);
    }
    
    let current = root;
    while (true) {
        if (val < current.val) {
            if (current.left === null) {
                current.left = new TreeNode(val);
                return current.left;
            }
            current = current.left;
        } else {
            if (current.right === null) {
                current.right = new TreeNode(val);
                return current.right;
            }
            current = current.right;
        }
    }
}

// Insert maintaining balance (AVL tree concept)
function insertBalanced(root, val) {
    // After normal BST insertion
    root = insertIntoBST(root, val);
    
    // Check and rebalance if needed
    // (Requires height tracking and rotations)
    return rebalance(root);
}

// Insert with validation
function insertWithValidation(root, val, min = -Infinity, max = Infinity) {
    if (val <= min || val >= max) {
        throw new Error("Invalid insertion - breaks BST property");
    }
    
    if (root === null) {
        return new TreeNode(val);
    }
    
    if (val < root.val) {
        root.left = insertWithValidation(root.left, val, min, root.val);
    } else {
        root.right = insertWithValidation(root.right, val, root.val, max);
    }
    
    return root;
}
```

## 🔗 Related Problems

- **Delete Node in BST** - More complex removal operation
- **Search in BST** - Similar traversal pattern
- **Validate BST** - Verify BST property
- **Convert Sorted Array to BST** - Build balanced BST

## 💡 Key Takeaways

1. **BST Property**: Always maintain left < root < right
2. **Recursive vs Iterative**: Trade-off between simplicity and space
3. **Empty Tree Edge Case**: Always handle null root
4. **Return Root**: Important for tree modifications
5. **Duplicate Handling**: Decide policy upfront

BST insertion is fundamental for understanding tree operations and maintaining sorted data structures!
