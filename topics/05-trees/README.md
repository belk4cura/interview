# 🌳 Trees & Binary Trees

## Overview
Trees are hierarchical data structures with nodes connected by edges. Binary trees (max 2 children per node) are the most common in interviews, forming the basis for many advanced data structures.

## Key Concepts

### Tree Types
- **Binary Tree**: Each node has at most 2 children
- **Binary Search Tree (BST)**: Left < Parent < Right
- **Balanced Trees**: AVL, Red-Black (height = O(log n))
- **Complete Binary Tree**: All levels full except possibly last
- **Perfect Binary Tree**: All internal nodes have 2 children
- **Heap**: Complete tree with heap property

### Tree Properties
| Property | Formula/Value |
|----------|--------------|
| Max nodes at level i | 2^i |
| Max nodes in tree of height h | 2^(h+1) - 1 |
| Min height of n nodes | floor(log₂n) |
| Max height of n nodes | n - 1 |

## Common Patterns

### Traversal Patterns
1. **DFS Traversals**
   - Preorder: Root → Left → Right
   - Inorder: Left → Root → Right (BST gives sorted)
   - Postorder: Left → Right → Root

2. **BFS Traversal**
   - Level-order: Level by level, left to right

### Recursive Patterns
```javascript
function treeRecursion(node) {
    if (!node) return; // Base case
    
    // Preorder processing
    processNode(node);
    
    treeRecursion(node.left);
    // Inorder processing
    treeRecursion(node.right);
    // Postorder processing
}
```

## Problems in This Section

### Basic
- [BST Insertion](problems/bst-insertion.md) - Add nodes to BST
- [Tree Height](problems/tree-height.md) - Calculate max depth
- [Tree Traversals](problems/tree-traversals.md) - All traversal types

### Intermediate
- [Lowest Common Ancestor](problems/lowest-common-ancestor.md) - Find LCA
- [Swap Nodes](problems/swap-nodes.md) - Modify tree structure
- [Min Heap Operations](problems/min-heap-operations.md) - Heap basics

### Advanced
- [Binary Tree Max Path Sum](problems/binary-tree-max-path-sum.md) - Complex path finding
- [Serialize/Deserialize Tree](problems/serialize-deserialize-tree.md) - Tree encoding
- [Find Duplicate Subtrees](problems/find-duplicate-subtrees.md) - Subtree matching

## Implementation Templates

```javascript
// Basic Tree Node
class TreeNode {
    constructor(val = 0, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

// BST Operations
class BST {
    insert(root, val) {
        if (!root) return new TreeNode(val);
        
        if (val < root.val) {
            root.left = this.insert(root.left, val);
        } else {
            root.right = this.insert(root.right, val);
        }
        return root;
    }
    
    search(root, val) {
        if (!root || root.val === val) return root;
        
        if (val < root.val) {
            return this.search(root.left, val);
        }
        return this.search(root.right, val);
    }
    
    findMin(root) {
        while (root.left) {
            root = root.left;
        }
        return root;
    }
    
    delete(root, val) {
        if (!root) return null;
        
        if (val < root.val) {
            root.left = this.delete(root.left, val);
        } else if (val > root.val) {
            root.right = this.delete(root.right, val);
        } else {
            // Node with 0 or 1 child
            if (!root.left) return root.right;
            if (!root.right) return root.left;
            
            // Node with 2 children
            const minRight = this.findMin(root.right);
            root.val = minRight.val;
            root.right = this.delete(root.right, minRight.val);
        }
        return root;
    }
}
```

### Traversal Implementations

```javascript
// DFS Traversals
function preorder(root, result = []) {
    if (!root) return result;
    result.push(root.val);
    preorder(root.left, result);
    preorder(root.right, result);
    return result;
}

function inorder(root, result = []) {
    if (!root) return result;
    inorder(root.left, result);
    result.push(root.val);
    inorder(root.right, result);
    return result;
}

function postorder(root, result = []) {
    if (!root) return result;
    postorder(root.left, result);
    postorder(root.right, result);
    result.push(root.val);
    return result;
}

// BFS Level-Order
function levelOrder(root) {
    if (!root) return [];
    
    const result = [];
    const queue = [root];
    
    while (queue.length) {
        const levelSize = queue.length;
        const level = [];
        
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

// Iterative Inorder (using stack)
function inorderIterative(root) {
    const result = [];
    const stack = [];
    let current = root;
    
    while (current || stack.length) {
        while (current) {
            stack.push(current);
            current = current.left;
        }
        current = stack.pop();
        result.push(current.val);
        current = current.right;
    }
    
    return result;
}
```

## Advanced Techniques

### Morris Traversal (O(1) space)
```javascript
function morrisInorder(root) {
    const result = [];
    let current = root;
    
    while (current) {
        if (!current.left) {
            result.push(current.val);
            current = current.right;
        } else {
            // Find inorder predecessor
            let predecessor = current.left;
            while (predecessor.right && predecessor.right !== current) {
                predecessor = predecessor.right;
            }
            
            if (!predecessor.right) {
                predecessor.right = current;
                current = current.left;
            } else {
                predecessor.right = null;
                result.push(current.val);
                current = current.right;
            }
        }
    }
    
    return result;
}
```

### Tree Diameter
```javascript
function diameter(root) {
    let maxDiameter = 0;
    
    function height(node) {
        if (!node) return 0;
        
        const leftHeight = height(node.left);
        const rightHeight = height(node.right);
        
        maxDiameter = Math.max(maxDiameter, leftHeight + rightHeight);
        
        return Math.max(leftHeight, rightHeight) + 1;
    }
    
    height(root);
    return maxDiameter;
}
```

## Tips for Success

1. **Master recursion** - Trees are naturally recursive
2. **Draw examples** - Visualize the tree structure
3. **Consider edge cases** - Empty tree, single node, skewed tree
4. **Track return values** - What each recursion returns
5. **Think about traversal order** - When to process the node
6. **Use helper functions** - Separate concerns

## Common Interview Questions

- Is this tree balanced?
- Find the LCA of two nodes
- Serialize and deserialize a tree
- Maximum path sum in tree
- Validate if tree is BST
- Convert BST to sorted doubly linked list

## When to Use Trees

✅ **Perfect for:**
- Hierarchical data (file systems, org charts)
- Searching with BST (O(log n) average)
- Priority queues (heaps)
- Expression parsing
- Decision trees

❌ **Avoid when:**
- Need constant-time access by index
- Data has no hierarchical relationship
- Frequent rebalancing is costly
