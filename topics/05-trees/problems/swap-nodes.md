# Swap Nodes in Binary Tree

## 🎯 What Are We Trying To Solve?

Imagine a binary tree where you can swap the left and right children of nodes at specific depths. It's like rearranging branches on a tree at certain heights - all branches at depth k, 2k, 3k, etc. get their left and right sides swapped!

## 📝 The Problem

Given a binary tree represented by an array of node connections, perform swaps at all depths that are multiples of a given value k. After each swap operation, output the in-order traversal of the modified tree.

```
Example:
Tree:     1         Swap at depth k=2
         / \
        2   3       Nodes at depth 2: [2, 3]
       /            Swap their children
      4             
                    Result:    1
                             / \
                            3   2
                               /
                              4
```

## 💻 The Code

```javascript
/**
 * Node class for tree representation
 */
class TreeNode {
    constructor(value, depth = 1) {
        this.value = value;
        this.left = null;
        this.right = null;
        this.depth = depth;
    }
}

/**
 * Build tree from array representation
 * @param {number[][]} indices - Array of [left, right] child indices
 * @returns {TreeNode} - Root of the tree
 */
function buildTree(indices) {
    const n = indices.length;
    const nodes = new Array(n + 1);
    
    // Create all nodes
    for (let i = 1; i <= n; i++) {
        nodes[i] = new TreeNode(i);
    }
    
    // Build connections using BFS to assign depths
    const queue = [nodes[1]];
    let nodeIndex = 1;
    
    while (queue.length > 0 && nodeIndex <= n) {
        const current = queue.shift();
        const [leftIndex, rightIndex] = indices[nodeIndex - 1];
        
        // Add left child
        if (leftIndex !== -1) {
            current.left = nodes[leftIndex];
            nodes[leftIndex].depth = current.depth + 1;
            queue.push(nodes[leftIndex]);
        }
        
        // Add right child
        if (rightIndex !== -1) {
            current.right = nodes[rightIndex];
            nodes[rightIndex].depth = current.depth + 1;
            queue.push(nodes[rightIndex]);
        }
        
        nodeIndex++;
    }
    
    return nodes[1]; // Return root
}

/**
 * Swap nodes at depths that are multiples of k
 * @param {TreeNode} root - Root of tree
 * @param {number} k - Swap at depths k, 2k, 3k, etc.
 */
function swapNodes(root, k) {
    if (!root) return;
    
    const queue = [root];
    const nodesByDepth = new Map();
    
    // Group nodes by depth
    while (queue.length > 0) {
        const node = queue.shift();
        
        if (!nodesByDepth.has(node.depth)) {
            nodesByDepth.set(node.depth, []);
        }
        nodesByDepth.get(node.depth).push(node);
        
        if (node.left) queue.push(node.left);
        if (node.right) queue.push(node.right);
    }
    
    // Swap at depths that are multiples of k
    for (const [depth, nodes] of nodesByDepth) {
        if (depth % k === 0) {
            for (const node of nodes) {
                // Swap left and right children
                [node.left, node.right] = [node.right, node.left];
            }
        }
    }
}

/**
 * Perform in-order traversal
 * @param {TreeNode} root - Root of tree
 * @returns {number[]} - In-order traversal result
 */
function inorderTraversal(root) {
    const result = [];
    
    function traverse(node) {
        if (!node) return;
        
        traverse(node.left);
        result.push(node.value);
        traverse(node.right);
    }
    
    traverse(root);
    return result;
}

/**
 * Alternative: Build tree using array indices (like original problem)
 */
function buildTreeFromIndices(indices) {
    const n = indices.length;
    if (n === 0) return null;
    
    // Create nodes array where index matches node value
    const nodes = [null]; // 0-index unused
    for (let i = 1; i <= n; i++) {
        nodes.push({ value: i, left: null, right: null, depth: 0 });
    }
    
    // Build tree structure
    nodes[1].depth = 1; // Root depth
    
    for (let i = 1; i <= n; i++) {
        const [leftIdx, rightIdx] = indices[i - 1];
        
        if (leftIdx !== -1) {
            nodes[i].left = nodes[leftIdx];
            nodes[leftIdx].depth = nodes[i].depth + 1;
        }
        
        if (rightIdx !== -1) {
            nodes[i].right = nodes[rightIdx];
            nodes[rightIdx].depth = nodes[i].depth + 1;
        }
    }
    
    return nodes[1];
}

/**
 * Swap and traverse multiple times
 * @param {number[][]} indices - Tree structure
 * @param {number[]} queries - List of k values for swapping
 * @returns {number[][]} - In-order traversals after each swap
 */
function swapAndTraverse(indices, queries) {
    const root = buildTree(indices);
    const results = [];
    
    for (const k of queries) {
        swapNodes(root, k);
        results.push(inorderTraversal(root));
    }
    
    return results;
}

/**
 * Optimized swap using recursion
 */
function swapNodesRecursive(root, k, currentDepth = 1) {
    if (!root) return;
    
    // Swap if current depth is multiple of k
    if (currentDepth % k === 0) {
        [root.left, root.right] = [root.right, root.left];
    }
    
    // Recurse to children
    swapNodesRecursive(root.left, k, currentDepth + 1);
    swapNodesRecursive(root.right, k, currentDepth + 1);
}

// Test cases
const indices1 = [
    [2, 3],   // Node 1: left=2, right=3
    [-1, -1], // Node 2: no children
    [-1, -1]  // Node 3: no children
];
const queries1 = [1, 1];

const results1 = swapAndTraverse(indices1, queries1);
console.log(results1);
// After swap at k=1: [3, 1, 2]
// After another swap at k=1: [2, 1, 3]

const indices2 = [
    [2, 3],
    [4, -1],
    [5, -1],
    [6, -1],
    [7, 8],
    [-1, 9],
    [-1, -1],
    [10, 11],
    [-1, -1],
    [-1, -1],
    [-1, -1]
];
const queries2 = [2, 4];

const results2 = swapAndTraverse(indices2, queries2);
console.log(results2);
```

## 🎨 Step-by-Step Walkthrough

Let's trace through swapping nodes at depth k=2:

```
Initial Tree:
       1 (depth=1)
      / \
     2   3 (depth=2)
    /   /
   4   5 (depth=3)

Swap at k=2:
- Nodes at depth 2: [2, 3]
- Swap their children:
  - Node 2: left=4, right=null → left=null, right=4
  - Node 3: left=5, right=null → left=null, right=5

Result:
       1
      / \
     2   3
      \   \
       4   5

In-order: 1, 2, 4, 3, 5
```

## 📊 Visual Understanding

```
Swapping Process:

Original:          After k=1:        After k=1 again:
    1                  1                   1
   / \                / \                 / \
  2   3              3   2               2   3
 /                       /               /
4                       4               4

Depths affected by k=2:
       1 (d=1)
      / \
     2   3 (d=2) ← Swap these nodes' children
    / \ / \
   4  5 6  7 (d=3)

After swap:
       1
      / \
     2   3
    / \ / \
   5  4 7  6  ← Children swapped
```

## ⏱️ Complexity Analysis

- **Time Complexity:**
  - Building tree: O(n)
  - Each swap operation: O(n) to traverse all nodes
  - Total for q queries: O(n × q)
  
- **Space Complexity:** O(n) for storing the tree

## 🚫 Common Mistakes

1. **Incorrect depth calculation**
   ```javascript
   // Wrong: Not updating depth correctly
   node.left = nodes[leftIdx];
   
   // Right: Set depth when creating connection
   node.left = nodes[leftIdx];
   nodes[leftIdx].depth = node.depth + 1;
   ```

2. **Swapping at wrong depths**
   ```javascript
   // Wrong: Only swapping at depth k
   if (depth === k) { swap(); }
   
   // Right: Swap at k, 2k, 3k, etc.
   if (depth % k === 0) { swap(); }
   ```

3. **Not preserving tree after swap**
   ```javascript
   // Wrong: Creating new nodes
   node.left = new TreeNode(rightValue);
   
   // Right: Swap references
   [node.left, node.right] = [node.right, node.left];
   ```

## 🔄 Variations

```javascript
// Swap only specific subtrees
function swapSubtree(root, targetValue, k) {
    function findAndSwap(node, depth = 1) {
        if (!node) return false;
        
        if (node.value === targetValue) {
            swapNodesRecursive(node, k, depth);
            return true;
        }
        
        return findAndSwap(node.left, depth + 1) || 
               findAndSwap(node.right, depth + 1);
    }
    
    findAndSwap(root);
}

// Swap based on custom condition
function swapConditional(root, condition) {
    function traverse(node, depth = 1) {
        if (!node) return;
        
        if (condition(node, depth)) {
            [node.left, node.right] = [node.right, node.left];
        }
        
        traverse(node.left, depth + 1);
        traverse(node.right, depth + 1);
    }
    
    traverse(root);
}

// Track swap history
function swapWithHistory(root, k) {
    const history = [];
    
    function swap(node, depth = 1) {
        if (!node) return;
        
        if (depth % k === 0) {
            history.push({
                node: node.value,
                depth: depth,
                action: 'swap'
            });
            [node.left, node.right] = [node.right, node.left];
        }
        
        swap(node.left, depth + 1);
        swap(node.right, depth + 1);
    }
    
    swap(root);
    return history;
}
```

## 🔗 Related Problems

- **Invert Binary Tree** - Swap all nodes
- **Symmetric Tree** - Check if tree is mirror
- **Binary Tree Level Order Traversal** - Process by levels
- **Reverse Odd Levels of Binary Tree** - Conditional swapping

## 💡 Key Takeaways

1. **Depth Tracking**: Essential for determining which nodes to swap
2. **Reference Swapping**: Swap pointers, not values
3. **Multiple of k**: Remember to swap at k, 2k, 3k, etc.
4. **Tree Traversal**: In-order gives sorted sequence for BST
5. **BFS vs DFS**: Both work for finding nodes at specific depths

This problem combines tree traversal with conditional modifications - great for understanding tree manipulation!
