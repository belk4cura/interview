# Lowest Common Ancestor

## 🎯 What Are We Trying To Solve?

Finding the lowest common ancestor is like finding the nearest common relative of two family members in a family tree - the closest shared ancestor.

## 📝 The Problem

Given a binary tree and two nodes, find their lowest common ancestor (LCA) - the deepest node that has both nodes as descendants.

```
Example:
       3
      / \
     5   1
    / \ / \
   6  2 0  8
     / \
    7   4

LCA(5, 1) = 3
LCA(5, 4) = 5
```

## 💻 The Code

```javascript
class TreeNode {
    constructor(val, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

/**
 * LCA in Binary Tree (nodes exist)
 * Time: O(n), Space: O(h)
 */
function lowestCommonAncestor(root, p, q) {
    // Base case
    if (!root || root === p || root === q) {
        return root;
    }
    
    // Search in subtrees
    const left = lowestCommonAncestor(root.left, p, q);
    const right = lowestCommonAncestor(root.right, p, q);
    
    // If both found in different subtrees, root is LCA
    if (left && right) return root;
    
    // Otherwise return the non-null result
    return left || right;
}

/**
 * LCA in Binary Search Tree
 * Time: O(h), Space: O(1)
 */
function lowestCommonAncestorBST(root, p, q) {
    while (root) {
        if (p.val < root.val && q.val < root.val) {
            root = root.left;
        } else if (p.val > root.val && q.val > root.val) {
            root = root.right;
        } else {
            return root;
        }
    }
    return null;
}

/**
 * LCA with parent pointers
 * Time: O(h), Space: O(h)
 */
function lowestCommonAncestorWithParent(p, q) {
    const ancestors = new Set();
    
    // Add all ancestors of p
    while (p) {
        ancestors.add(p);
        p = p.parent;
    }
    
    // Find first common ancestor with q
    while (q) {
        if (ancestors.has(q)) return q;
        q = q.parent;
    }
    
    return null;
}
```

## 🎨 Key Insights

- **Recursive Pattern**: If nodes on different sides, current is LCA
- **BST Property**: Use value comparison for optimization
- **Parent Pointers**: Alternative approach using hash set

## ⏱️ Complexity

- Binary Tree: O(n) time, O(h) space
- BST: O(h) time, O(1) space iterative
