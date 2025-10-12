# Binary Tree Maximum Path Sum

## 🎯 What Are We Trying To Solve?

Imagine you're hiking through a mountain trail system where each junction has a value (positive for scenic views, negative for difficult terrain). You want to find the path with the highest total value. The catch: you can go through a junction at most once, and your path doesn't need to start at the top or end at the bottom - it can be ANY connected path!

## 📝 The Problem

Given a binary tree where each node has a value, find the maximum sum of any path. A path can start and end at any nodes but must follow parent-child connections.

```
Example Tree:
       -10
       /  \
      9    20
          /  \
         15   7

Maximum Path: 15 → 20 → 7 = 42
```

## 🧠 How to Think About This

The key insight: At each node, we have choices:
1. Include just this node
2. Include this node + best path from left child
3. Include this node + best path from right child  
4. Include this node as a "bridge" connecting left and right paths

### Visual Path Types
```
Type 1: Single Node      Type 2: Going Through
     [5]                       [5]
                              /   \
                            [3]   [7]
   Sum = 5                 Sum = 3+5+7 = 15

Type 3: Left Path        Type 4: Right Path
      [5]                      [5]
     /                           \
   [3]                           [7]
   Sum = 3+5 = 8               Sum = 5+7 = 12
```

## 💻 The Code

```javascript
function maxPathSum(root) {
    // Track the maximum sum we've seen
    let maxSum = -Infinity;
    
    // Helper function returns max gain from this node down
    function maxGain(node) {
        // Base case: empty node contributes 0
        if (!node) return 0;
        
        // Recursively get max gain from left and right
        // Use Math.max with 0 to ignore negative paths
        const leftGain = Math.max(maxGain(node.left), 0);
        const rightGain = Math.max(maxGain(node.right), 0);
        
        // Path that goes THROUGH this node (can't extend further up)
        // This is: left path + node + right path
        const pathThroughNode = node.val + leftGain + rightGain;
        
        // Update our global maximum
        maxSum = Math.max(maxSum, pathThroughNode);
        
        // Return max gain if path continues through this node
        // Can only pick ONE side to continue up
        return node.val + Math.max(leftGain, rightGain);
    }
    
    maxGain(root);
    return maxSum;
}

// Test with examples
class TreeNode {
    constructor(val, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

// Example 1: [-10, 9, 20, null, null, 15, 7]
const tree1 = new TreeNode(-10,
    new TreeNode(9),
    new TreeNode(20,
        new TreeNode(15),
        new TreeNode(7)
    )
);
console.log(maxPathSum(tree1)); // 42 (path: 15→20→7)

// Example 2: Single node
const tree2 = new TreeNode(-3);
console.log(maxPathSum(tree2)); // -3

// Example 3: All positive
const tree3 = new TreeNode(1,
    new TreeNode(2),
    new TreeNode(3)
);
console.log(maxPathSum(tree3)); // 6 (path: 2→1→3)
```

## 🎨 Step-by-Step Walkthrough

Let's trace through the example tree:

```
Tree:    -10
         /  \
        9    20
            /  \
           15   7

Step 1: Process leaf 9
- leftGain = 0 (no left child)
- rightGain = 0 (no right child)
- pathThrough = 9 + 0 + 0 = 9
- maxSum = 9
- return 9 (can extend up)

Step 2: Process leaf 15
- pathThrough = 15
- maxSum = 15
- return 15

Step 3: Process leaf 7
- pathThrough = 7
- maxSum = 15 (no change)
- return 7

Step 4: Process node 20
- leftGain = 15 (from step 2)
- rightGain = 7 (from step 3)
- pathThrough = 20 + 15 + 7 = 42
- maxSum = 42 ✓
- return 20 + max(15,7) = 35

Step 5: Process root -10
- leftGain = 9 (from step 1)
- rightGain = 35 (from step 4)
- pathThrough = -10 + 9 + 35 = 34
- maxSum = 42 (no change)
- return doesn't matter (root)

Final answer: 42
```

## 📊 Why This Works

### The Two Values We Track

1. **Global Maximum** (`maxSum`): The best path found anywhere
2. **Return Value** (`maxGain`): Best path that can extend upward

```
At each node:
┌─────────────────────────────────┐
│ Can this be a path peak?        │
│ (Check left + node + right)     │
│ Update maxSum if better         │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│ What do I tell my parent?       │
│ (node + best single child path) │
│ This can extend upward          │
└─────────────────────────────────┘
```

## ⏱️ Complexity Analysis

- **Time: O(n)** - Visit each node exactly once
- **Space: O(h)** - Recursion stack depth = tree height
  - Best case (balanced): O(log n)
  - Worst case (skewed): O(n)

## 🚫 Common Mistakes

1. **Forgetting Negative Values**
   ```javascript
   // Wrong: Always include children
   const leftGain = maxGain(node.left);
   
   // Right: Ignore negative contributions
   const leftGain = Math.max(maxGain(node.left), 0);
   ```

2. **Returning the Global Maximum**
   ```javascript
   // Wrong: Return the global max
   return maxSum;
   
   // Right: Return max gain through this node
   return node.val + Math.max(leftGain, rightGain);
   ```

3. **Not Considering Single Node Paths**
   ```javascript
   // A single node can be the maximum path!
   // Tree: [-3] → Answer: -3 (not 0!)
   ```

## 🔄 Alternative Approaches

### Approach 2: Track Path Nodes
```javascript
function maxPathSumWithPath(root) {
    let maxSum = -Infinity;
    let maxPath = [];
    
    function traverse(node, currentPath) {
        if (!node) return { sum: 0, path: [] };
        
        const left = traverse(node.left, [...currentPath, node.val]);
        const right = traverse(node.right, [...currentPath, node.val]);
        
        // Calculate all possible paths through this node
        const paths = [
            { sum: node.val, path: [node.val] },
            { sum: node.val + left.sum, path: [node.val, ...left.path] },
            { sum: node.val + right.sum, path: [node.val, ...right.path] },
            { sum: node.val + left.sum + right.sum, 
              path: [...left.path.reverse(), node.val, ...right.path] }
        ];
        
        // Update global maximum
        for (const p of paths) {
            if (p.sum > maxSum) {
                maxSum = p.sum;
                maxPath = p.path;
            }
        }
        
        // Return best extensible path
        const better = left.sum > right.sum ? left : right;
        return {
            sum: node.val + Math.max(0, better.sum),
            path: better.sum > 0 ? [node.val, ...better.path] : [node.val]
        };
    }
    
    traverse(root, []);
    return { maxSum, maxPath };
}
```

## 🔗 Related Problems

- **Binary Tree Maximum Path Product** - Same concept with multiplication
- **Diameter of Binary Tree** - Maximum path length between any two nodes
- **House Robber III** - Maximum sum with constraints
- **Path Sum III** - Count paths that sum to target

## 💡 Key Takeaways

1. **Think Recursively**: What can each subtree tell us?
2. **Track Two Things**: Global answer vs. what to return to parent
3. **Handle Negatives**: Use `Math.max(0, ...)` to exclude bad paths
4. **Every Node Matters**: Even single nodes can be the answer

This pattern of "calculate locally, update globally" appears in many tree problems!
