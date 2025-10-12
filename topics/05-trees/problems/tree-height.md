# Height of a Binary Tree

## 🎯 What Are We Trying To Solve?

Finding the height of a binary tree is like measuring how many floors a building has - you need to find the longest path from the ground floor (root) to the highest apartment (leaf). The height represents the maximum number of edges from root to any leaf.

## 📝 The Problem

Given the root of a binary tree, find its height. The height is defined as:
- The number of edges on the longest path from root to a leaf
- An empty tree has height -1 (some define it as 0)
- A single node has height 0 (or 1, depending on definition)

```
Example:
       1           Height = 3
      / \          (Path: 1→2→4→7)
     2   3
    / \
   4   5
  /
 7
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
 * Calculate height of binary tree (edges definition)
 * Height = number of edges on longest path
 * @param {TreeNode} root - Root of tree
 * @returns {number} - Height of tree
 */
function height(root) {
    // Base case: empty tree or leaf node
    if (root === null) {
        return -1;  // Empty tree has height -1
    }
    
    if (root.left === null && root.right === null) {
        return 0;  // Leaf node has height 0
    }
    
    // Recursively find height of subtrees
    const leftHeight = height(root.left);
    const rightHeight = height(root.right);
    
    // Height is max of subtrees plus 1
    return Math.max(leftHeight, rightHeight) + 1;
}

/**
 * Alternative: Height counting nodes (common variation)
 * Height = number of nodes on longest path
 */
function heightNodes(root) {
    if (root === null) {
        return 0;  // Empty tree has 0 nodes
    }
    
    const leftHeight = heightNodes(root.left);
    const rightHeight = heightNodes(root.right);
    
    return Math.max(leftHeight, rightHeight) + 1;
}

/**
 * Calculate depth of a specific node
 * Depth = edges from root to node
 */
function nodeDepth(root, target, depth = 0) {
    if (root === null) {
        return -1;  // Node not found
    }
    
    if (root === target) {
        return depth;
    }
    
    // Search in left subtree
    const leftDepth = nodeDepth(root.left, target, depth + 1);
    if (leftDepth !== -1) {
        return leftDepth;
    }
    
    // Search in right subtree
    return nodeDepth(root.right, target, depth + 1);
}

/**
 * Iterative height using level-order traversal
 */
function heightIterative(root) {
    if (root === null) return -1;
    
    const queue = [root];
    let height = -1;
    
    while (queue.length > 0) {
        const levelSize = queue.length;
        height++;
        
        // Process all nodes at current level
        for (let i = 0; i < levelSize; i++) {
            const node = queue.shift();
            
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
    }
    
    return height;
}

/**
 * Find minimum depth (shortest path to leaf)
 */
function minDepth(root) {
    if (root === null) return 0;
    
    // If one subtree is null, use the other
    if (root.left === null) {
        return minDepth(root.right) + 1;
    }
    if (root.right === null) {
        return minDepth(root.left) + 1;
    }
    
    // Both subtrees exist, take minimum
    return Math.min(minDepth(root.left), minDepth(root.right)) + 1;
}

/**
 * Check if tree is balanced (height difference ≤ 1)
 */
function isBalanced(root) {
    function checkHeight(node) {
        if (node === null) return 0;
        
        const leftHeight = checkHeight(node.left);
        if (leftHeight === -1) return -1;  // Left subtree unbalanced
        
        const rightHeight = checkHeight(node.right);
        if (rightHeight === -1) return -1;  // Right subtree unbalanced
        
        // Check current node balance
        if (Math.abs(leftHeight - rightHeight) > 1) {
            return -1;  // Current node unbalanced
        }
        
        return Math.max(leftHeight, rightHeight) + 1;
    }
    
    return checkHeight(root) !== -1;
}

// Test cases
const root1 = new TreeNode(1,
    new TreeNode(2,
        new TreeNode(4,
            new TreeNode(7)),
        new TreeNode(5)),
    new TreeNode(3)
);
console.log(height(root1));  // 3

const root2 = new TreeNode(1);
console.log(height(root2));  // 0

const root3 = null;
console.log(height(root3));  // -1

const root4 = new TreeNode(1,
    new TreeNode(2),
    new TreeNode(3,
        null,
        new TreeNode(4,
            null,
            new TreeNode(5)))
);
console.log(height(root4));  // 3
console.log(isBalanced(root4));  // false
```

## 🎨 Step-by-Step Walkthrough

Let's trace through finding the height of this tree:

```
       1
      / \
     2   3
    / \
   4   5

Step 1: height(1)
- Not null, has children
- Calculate height(2) and height(3)

Step 2: height(2)
- Not null, has children
- Calculate height(4) and height(5)

Step 3: height(4)
- Leaf node → return 0

Step 4: height(5)
- Leaf node → return 0

Step 5: Back to height(2)
- leftHeight = 0, rightHeight = 0
- Return max(0, 0) + 1 = 1

Step 6: height(3)
- Leaf node → return 0

Step 7: Back to height(1)
- leftHeight = 1, rightHeight = 0
- Return max(1, 0) + 1 = 2

Final height: 2
```

## 📊 Visual Understanding

```
Height vs Depth:

       A (depth=0)        Height of tree = 3
      / \
     B   C (depth=1)      Height from A = 3
    / \                   Height from B = 2
   D   E (depth=2)        Height from D = 1
  /                       Height from F = 0
 F (depth=3)

Height: Maximum distance DOWN from a node to a leaf
Depth:  Distance UP from a node to the root
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(n)
  - Visit every node exactly once
  
- **Space Complexity:** 
  - Recursive: O(h) where h is height (call stack)
  - Iterative: O(w) where w is max width (queue size)

## 🚫 Common Mistakes

1. **Confusing height definitions**
   ```javascript
   // Wrong: Mixing edges and nodes
   if (root === null) return 0;  // Should be -1 for edges
   
   // Be consistent with your definition:
   // Edges: null → -1, leaf → 0
   // Nodes: null → 0, leaf → 1
   ```

2. **Not handling null children properly**
   ```javascript
   // Wrong: Doesn't check for null
   return Math.max(height(root.left), height(root.right)) + 1;
   
   // Right: Base case handles null
   if (root === null) return -1;
   ```

3. **Off-by-one errors**
   ```javascript
   // Wrong: Forgetting to add 1
   return Math.max(leftHeight, rightHeight);
   
   // Right: Add 1 for current level
   return Math.max(leftHeight, rightHeight) + 1;
   ```

## 🔄 Variations

```javascript
// Maximum depth (same as height with nodes definition)
function maxDepth(root) {
    if (root === null) return 0;
    return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
}

// Diameter of tree (longest path between any two nodes)
function diameter(root) {
    let maxDiameter = 0;
    
    function height(node) {
        if (node === null) return 0;
        
        const left = height(node.left);
        const right = height(node.right);
        
        // Update diameter if path through current node is longer
        maxDiameter = Math.max(maxDiameter, left + right);
        
        return Math.max(left, right) + 1;
    }
    
    height(root);
    return maxDiameter;
}

// All paths from root to leaves
function allPaths(root) {
    const paths = [];
    
    function dfs(node, path) {
        if (node === null) return;
        
        path.push(node.val);
        
        if (node.left === null && node.right === null) {
            paths.push([...path]);
        } else {
            dfs(node.left, path);
            dfs(node.right, path);
        }
        
        path.pop();
    }
    
    dfs(root, []);
    return paths;
}
```

## 🔗 Related Problems

- **Maximum Depth of Binary Tree** - Same concept, different terminology
- **Balanced Binary Tree** - Uses height to check balance
- **Diameter of Binary Tree** - Longest path between any nodes
- **Minimum Depth of Binary Tree** - Shortest path to leaf

## 💡 Key Takeaways

1. **Clarify Definition**: Height as edges vs nodes
2. **Recursive Pattern**: Max of subtrees + 1
3. **Base Cases**: Handle null and leaf nodes
4. **Height vs Depth**: Height goes down, depth goes up
5. **Applications**: Balance checking, tree properties

Understanding tree height is fundamental for analyzing tree algorithms and their complexity!
