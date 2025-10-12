# Tree Traversals (Preorder, Inorder, Postorder, Level-Order)

## 🎯 What Are We Trying To Solve?

Tree traversal is like exploring a building systematically - you need a strategy to visit every room exactly once. Different traversal orders give us different ways to process tree nodes, each useful for different purposes.

## 📝 The Problem

Implement the four main tree traversal methods:
1. **Preorder**: Root → Left → Right (process node before children)
2. **Inorder**: Left → Root → Right (gives sorted order in BST)
3. **Postorder**: Left → Right → Root (process children before node)
4. **Level-Order**: Level by level, left to right (BFS)

```
Example Tree:
       1
      / \
     2   3
    / \
   4   5

Preorder:    1, 2, 4, 5, 3
Inorder:     4, 2, 5, 1, 3
Postorder:   4, 5, 2, 3, 1
Level-Order: 1, 2, 3, 4, 5
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
 * PREORDER TRAVERSAL: Root → Left → Right
 */
// Recursive
function preorderRecursive(root) {
    const result = [];
    
    function traverse(node) {
        if (node === null) return;
        
        result.push(node.val);    // Process root
        traverse(node.left);       // Visit left subtree
        traverse(node.right);      // Visit right subtree
    }
    
    traverse(root);
    return result;
}

// Iterative using stack
function preorderIterative(root) {
    if (root === null) return [];
    
    const result = [];
    const stack = [root];
    
    while (stack.length > 0) {
        const node = stack.pop();
        result.push(node.val);
        
        // Push right first so left is processed first
        if (node.right) stack.push(node.right);
        if (node.left) stack.push(node.left);
    }
    
    return result;
}

/**
 * INORDER TRAVERSAL: Left → Root → Right
 */
// Recursive
function inorderRecursive(root) {
    const result = [];
    
    function traverse(node) {
        if (node === null) return;
        
        traverse(node.left);       // Visit left subtree
        result.push(node.val);     // Process root
        traverse(node.right);      // Visit right subtree
    }
    
    traverse(root);
    return result;
}

// Iterative using stack
function inorderIterative(root) {
    const result = [];
    const stack = [];
    let current = root;
    
    while (current !== null || stack.length > 0) {
        // Go to leftmost node
        while (current !== null) {
            stack.push(current);
            current = current.left;
        }
        
        // Current is null, pop from stack
        current = stack.pop();
        result.push(current.val);
        
        // Visit right subtree
        current = current.right;
    }
    
    return result;
}

/**
 * POSTORDER TRAVERSAL: Left → Right → Root
 */
// Recursive
function postorderRecursive(root) {
    const result = [];
    
    function traverse(node) {
        if (node === null) return;
        
        traverse(node.left);       // Visit left subtree
        traverse(node.right);      // Visit right subtree
        result.push(node.val);     // Process root
    }
    
    traverse(root);
    return result;
}

// Iterative using two stacks
function postorderIterative(root) {
    if (root === null) return [];
    
    const result = [];
    const stack1 = [root];
    const stack2 = [];
    
    // First stack for traversal
    while (stack1.length > 0) {
        const node = stack1.pop();
        stack2.push(node);
        
        if (node.left) stack1.push(node.left);
        if (node.right) stack1.push(node.right);
    }
    
    // Second stack has nodes in reverse postorder
    while (stack2.length > 0) {
        result.push(stack2.pop().val);
    }
    
    return result;
}

// Iterative using one stack (more complex)
function postorderOneStack(root) {
    if (root === null) return [];
    
    const result = [];
    const stack = [];
    let lastVisited = null;
    let current = root;
    
    while (current !== null || stack.length > 0) {
        if (current !== null) {
            stack.push(current);
            current = current.left;
        } else {
            const peekNode = stack[stack.length - 1];
            
            // If right child exists and not processed yet
            if (peekNode.right !== null && lastVisited !== peekNode.right) {
                current = peekNode.right;
            } else {
                result.push(peekNode.val);
                lastVisited = stack.pop();
            }
        }
    }
    
    return result;
}

/**
 * LEVEL-ORDER TRAVERSAL (BFS)
 */
function levelOrder(root) {
    if (root === null) return [];
    
    const result = [];
    const queue = [root];
    
    while (queue.length > 0) {
        const levelSize = queue.length;
        const currentLevel = [];
        
        for (let i = 0; i < levelSize; i++) {
            const node = queue.shift();
            currentLevel.push(node.val);
            
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        
        result.push(currentLevel);
    }
    
    return result;
}

// Level-order as flat array
function levelOrderFlat(root) {
    if (root === null) return [];
    
    const result = [];
    const queue = [root];
    
    while (queue.length > 0) {
        const node = queue.shift();
        result.push(node.val);
        
        if (node.left) queue.push(node.left);
        if (node.right) queue.push(node.right);
    }
    
    return result;
}

/**
 * MORRIS TRAVERSAL (O(1) space inorder)
 */
function morrisInorder(root) {
    const result = [];
    let current = root;
    
    while (current !== null) {
        if (current.left === null) {
            result.push(current.val);
            current = current.right;
        } else {
            // Find inorder predecessor
            let predecessor = current.left;
            while (predecessor.right !== null && predecessor.right !== current) {
                predecessor = predecessor.right;
            }
            
            if (predecessor.right === null) {
                // Make current the right child of predecessor
                predecessor.right = current;
                current = current.left;
            } else {
                // Revert the changes
                predecessor.right = null;
                result.push(current.val);
                current = current.right;
            }
        }
    }
    
    return result;
}

// Test all traversals
const root = new TreeNode(1,
    new TreeNode(2,
        new TreeNode(4),
        new TreeNode(5)),
    new TreeNode(3)
);

console.log("Preorder:", preorderRecursive(root));    // [1, 2, 4, 5, 3]
console.log("Inorder:", inorderRecursive(root));      // [4, 2, 5, 1, 3]
console.log("Postorder:", postorderRecursive(root));  // [4, 5, 2, 3, 1]
console.log("Level-order:", levelOrderFlat(root));    // [1, 2, 3, 4, 5]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through each traversal for this tree:

```
       1
      / \
     2   3
    / \
   4   5

PREORDER (Root → Left → Right):
Visit 1 → Process 1
Go left to 2 → Process 2
Go left to 4 → Process 4 (leaf)
Back to 2, go right to 5 → Process 5 (leaf)
Back to 1, go right to 3 → Process 3 (leaf)
Result: [1, 2, 4, 5, 3]

INORDER (Left → Root → Right):
Go left to 2, then left to 4
Process 4 (leftmost)
Back to 2 → Process 2
Go right to 5 → Process 5
Back to 1 → Process 1
Go right to 3 → Process 3
Result: [4, 2, 5, 1, 3]

POSTORDER (Left → Right → Root):
Go to leftmost 4 → Process 4
Back to 2, go right to 5 → Process 5
Back to 2 → Process 2
Back to 1, go right to 3 → Process 3
Back to 1 → Process 1
Result: [4, 5, 2, 3, 1]
```

## 📊 Visual Understanding

```
Traversal Paths:

PREORDER (DLR):          INORDER (LDR):
    1 ←start                 4 ←start
   ↙ ↘                      ↗
  2   3                    2
 ↙ ↘                      ↗ ↘
4   5                    1   5
                          ↘
Order: 1→2→4→5→3           3

POSTORDER (LRD):        LEVEL-ORDER:
    1 ←end              Level 0: [1]
   ↙ ↘                  Level 1: [2,3]
  2   3                 Level 2: [4,5]
 ↙ ↘
4   5 ←start
```

## ⏱️ Complexity Analysis

| Method | Time | Space (Recursive) | Space (Iterative) |
|--------|------|------------------|-------------------|
| Preorder | O(n) | O(h) | O(h) |
| Inorder | O(n) | O(h) | O(h) |
| Postorder | O(n) | O(h) | O(h) |
| Level-Order | O(n) | - | O(w) |
| Morris | O(n) | - | O(1) |

Where:
- n = number of nodes
- h = height of tree
- w = maximum width of tree

## 🚫 Common Mistakes

1. **Wrong order in recursive calls**
   ```javascript
   // Wrong: This is not preorder!
   traverse(node.left);
   result.push(node.val);
   traverse(node.right);
   
   // Right: Preorder is Root-Left-Right
   result.push(node.val);
   traverse(node.left);
   traverse(node.right);
   ```

2. **Stack order for iterative preorder**
   ```javascript
   // Wrong: Pushes left first
   if (node.left) stack.push(node.left);
   if (node.right) stack.push(node.right);
   
   // Right: Push right first (stack is LIFO)
   if (node.right) stack.push(node.right);
   if (node.left) stack.push(node.left);
   ```

## 🔄 Variations

```javascript
// Zigzag level-order traversal
function zigzagLevelOrder(root) {
    if (!root) return [];
    
    const result = [];
    const queue = [root];
    let leftToRight = true;
    
    while (queue.length > 0) {
        const levelSize = queue.length;
        const currentLevel = [];
        
        for (let i = 0; i < levelSize; i++) {
            const node = queue.shift();
            
            if (leftToRight) {
                currentLevel.push(node.val);
            } else {
                currentLevel.unshift(node.val);
            }
            
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        
        result.push(currentLevel);
        leftToRight = !leftToRight;
    }
    
    return result;
}

// Vertical order traversal
function verticalOrder(root) {
    if (!root) return [];
    
    const map = new Map();
    const queue = [[root, 0]];
    let minCol = 0, maxCol = 0;
    
    while (queue.length > 0) {
        const [node, col] = queue.shift();
        
        if (!map.has(col)) {
            map.set(col, []);
        }
        map.get(col).push(node.val);
        
        minCol = Math.min(minCol, col);
        maxCol = Math.max(maxCol, col);
        
        if (node.left) queue.push([node.left, col - 1]);
        if (node.right) queue.push([node.right, col + 1]);
    }
    
    const result = [];
    for (let i = minCol; i <= maxCol; i++) {
        result.push(map.get(i));
    }
    
    return result;
}
```

## 💡 Key Takeaways

1. **Remember the Orders**: DLR (Pre), LDR (In), LRD (Post)
2. **Recursive vs Iterative**: Trade simplicity for explicit stack control
3. **Stack vs Queue**: DFS uses stack, BFS uses queue
4. **BST Property**: Inorder traversal gives sorted order
5. **Space Optimization**: Morris traversal achieves O(1) space

Tree traversals are fundamental for processing tree data structures!
