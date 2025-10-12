# Find Duplicate Subtrees

## 🎯 What Are We Trying To Solve?

Imagine you're looking at a family tree and want to find all the branches that look exactly the same - same structure, same names, everything. That's what we're doing here with binary trees - finding all subtrees that are duplicates of each other!

## 📝 The Problem

Given the root of a binary tree, find all duplicate subtrees. For each duplicate subtree, you only need to return the root node of any one of them. Two trees are duplicates if they have the same structure and node values.

```
Example Tree:
        1
       / \
      2   3
     /   / \
    4   2   4
       /
      4

Duplicate subtrees:
- The subtree with root 2 (and child 4) appears twice
- The subtree with root 4 (leaf) appears three times
```

## 🧠 How to Think About This

The key insight: If we can create a unique "signature" for each subtree, we can use a hash map to track which signatures we've seen before!

### Visual Process
```
Serialize each subtree:
     2        →  "2,4,#,#,#"
    /
   4

     2        →  "2,4,#,#,#"  (Same! It's a duplicate)
    /
   4

   4 (leaf)   →  "4,#,#"  (Appears 3 times in tree)
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
 * Find all duplicate subtrees
 * Time: O(n) where n is number of nodes
 * Space: O(n) for the hash map and recursion stack
 */
function findDuplicateSubtrees(root) {
    const result = [];
    const subtreeMap = new Map();
    
    /**
     * Serialize subtree and track occurrences
     * Returns serialized string representation
     */
    function serialize(node) {
        // Base case: null node represented as '#'
        if (!node) return '#';
        
        // Create unique string for this subtree
        // Format: "value,leftSubtree,rightSubtree"
        const leftSerial = serialize(node.left);
        const rightSerial = serialize(node.right);
        const subtreeSerial = `${node.val},${leftSerial},${rightSerial}`;
        
        // Track how many times we've seen this exact subtree
        const count = (subtreeMap.get(subtreeSerial) || 0) + 1;
        subtreeMap.set(subtreeSerial, count);
        
        // If we've seen it exactly twice, add to result
        // (exactly twice to avoid adding same duplicate multiple times)
        if (count === 2) {
            result.push(node);
        }
        
        return subtreeSerial;
    }
    
    serialize(root);
    return result;
}

// Test with examples
// Tree:     1
//          / \
//         2   3
//        /   / \
//       4   2   4
//          /
//         4

const root = new TreeNode(1,
    new TreeNode(2,
        new TreeNode(4)
    ),
    new TreeNode(3,
        new TreeNode(2,
            new TreeNode(4)
        ),
        new TreeNode(4)
    )
);

const duplicates = findDuplicateSubtrees(root);
console.log(duplicates.map(node => node.val)); // [4, 2]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through the example tree:

```
Tree:     1
         / \
        2   3
       /   / \
      4   2   4
          /
         4

Processing order (post-order traversal):

Step 1: Serialize leaf 4 (leftmost)
- Serial: "4,#,#"
- Map: {"4,#,#": 1}

Step 2: Serialize node 2 (left subtree)
- Serial: "2,4,#,#,#"
- Map: {"4,#,#": 1, "2,4,#,#,#": 1}

Step 3: Serialize leaf 4 (under second 2)
- Serial: "4,#,#"
- Map: {"4,#,#": 2}
- Count = 2, add node 4 to result!

Step 4: Serialize node 2 (under 3)
- Serial: "2,4,#,#,#"
- Map: {"4,#,#": 2, "2,4,#,#,#": 2}
- Count = 2, add node 2 to result!

Step 5: Serialize leaf 4 (rightmost)
- Serial: "4,#,#"
- Map: {"4,#,#": 3}
- Count = 3, don't add (already added at count 2)

Step 6: Serialize node 3
- Serial: "3,2,4,#,#,#,4,#,#"
- Map: {..., "3,2,4,#,#,#,4,#,#": 1}

Step 7: Serialize root 1
- Serial: "1,2,4,#,#,#,3,2,4,#,#,#,4,#,#"
- Map: {..., "1,2,4,#,#,#,3,2,4,#,#,#,4,#,#": 1}

Result: [node4, node2] (the duplicate subtrees)
```

## 📊 Why This Works

### The Serialization Key

Each subtree gets a unique string representation:
- Same structure + same values = same string
- Different structure or values = different string

```
     2           vs        2
    / \                     \
   1   3                     3
                            /
                           1

"2,1,#,#,3,#,#"  vs  "2,#,3,1,#,#,#"
Different strings = Not duplicates!
```

### Why Count == 2?

```javascript
if (count === 2) {
    result.push(node);
}
```

- Count = 1: First occurrence, not a duplicate yet
- Count = 2: Second occurrence, it's a duplicate! Add it once
- Count > 2: More occurrences, but we already added it

## 🚫 Common Mistakes

1. **Adding Duplicates Multiple Times**
   ```javascript
   // Wrong: Adding every time we see a duplicate
   if (count >= 2) {
       result.push(node); // Would add same duplicate many times!
   }
   
   // Right: Add only when count equals 2
   if (count === 2) {
       result.push(node);
   }
   ```

2. **Wrong Serialization Format**
   ```javascript
   // Wrong: Can't distinguish structure
   const serial = `${node.val}${left}${right}`;
   // "123" could be "1,2,3" or "12,3" or "1,23"
   
   // Right: Use delimiters
   const serial = `${node.val},${left},${right}`;
   ```

3. **Not Handling Null Nodes**
   ```javascript
   // Wrong: Ignoring nulls loses structure info
   if (!node) return '';
   
   // Right: Mark nulls explicitly
   if (!node) return '#';
   ```

## 🔄 Alternative Approaches

### Approach 2: Using Tree Hashing
```javascript
function findDuplicateSubtreesHash(root) {
    const result = [];
    const seen = new Map();
    const duplicates = new Set();
    
    function hash(node) {
        if (!node) return 0;
        
        // Create hash from structure and values
        const tuple = `${hash(node.left)},${node.val},${hash(node.right)}`;
        
        if (seen.has(tuple)) {
            const treeId = seen.get(tuple);
            if (!duplicates.has(treeId)) {
                duplicates.add(treeId);
                result.push(node);
            }
            return treeId;
        }
        
        const newId = seen.size + 1;
        seen.set(tuple, newId);
        return newId;
    }
    
    hash(root);
    return result;
}
```

### Approach 3: Merkle Tree Style
```javascript
function findDuplicateSubtreesMerkle(root) {
    const result = [];
    const hashToCount = new Map();
    const hashToNode = new Map();
    
    function merkleHash(node) {
        if (!node) return 'null';
        
        const leftHash = merkleHash(node.left);
        const rightHash = merkleHash(node.right);
        
        // Create hash like in blockchain Merkle trees
        const currentHash = `(${leftHash}${node.val}${rightHash})`;
        
        const count = (hashToCount.get(currentHash) || 0) + 1;
        hashToCount.set(currentHash, count);
        
        if (count === 1) {
            hashToNode.set(currentHash, node);
        } else if (count === 2) {
            result.push(hashToNode.get(currentHash));
        }
        
        return currentHash;
    }
    
    merkleHash(root);
    return result;
}
```

## 🔗 Related Problems

- **Subtree of Another Tree** - Check if one tree is subtree of another
- **Same Tree** - Check if two trees are identical
- **Symmetric Tree** - Check if tree is mirror of itself
- **Serialize and Deserialize Binary Tree** - Similar serialization technique

## 💡 Key Takeaways

1. **Serialization Creates Unique IDs**: Convert structure to string for comparison
2. **Hash Maps Track Occurrences**: Perfect for finding duplicates
3. **Post-order Traversal**: Process children before parents
4. **Count Exactly 2**: Clever way to add each duplicate only once

This pattern of serializing structures for comparison is powerful and appears in many tree problems!
