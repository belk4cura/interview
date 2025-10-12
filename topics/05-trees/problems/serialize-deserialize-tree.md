# Serialize and Deserialize Binary Tree

## 🎯 What Are We Trying To Solve?

Imagine you need to save a family tree to a text file, then later rebuild the exact same tree from that file. Or think of it like taking apart a LEGO structure, writing down instructions, then rebuilding it exactly the same way. That's serialization (tree → string) and deserialization (string → tree)!

## 📝 The Problem

Design an algorithm to serialize a binary tree into a string and deserialize that string back to the original tree structure.

```
Original Tree:        Serialized:             Deserialized:
      1               "1,2,null,null,          1
     / \               3,4,null,null,          / \
    2   3              5,null,null"           2   3
       / \                                       / \
      4   5                                     4   5
```

## 🧠 How to Think About This

### Serialization Approaches

1. **Pre-order**: Root → Left → Right (easiest to deserialize)
2. **Level-order**: Level by level (BFS style)
3. **In-order**: Won't work alone (can't uniquely identify structure)

We'll use **pre-order** because when deserializing, we know the first value is always the root!

### Visual Process
```
Serialize (Pre-order):
      1
     / \
    2   3
   
Step 1: Process 1 → "1,"
Step 2: Go left to 2 → "1,2,"
Step 3: Left of 2 is null → "1,2,null,"
Step 4: Right of 2 is null → "1,2,null,null,"
Step 5: Go to 3 → "1,2,null,null,3,"
Step 6: Left of 3 is null → "1,2,null,null,3,null,"
Step 7: Right of 3 is null → "1,2,null,null,3,null,null"
```

## 💻 The Code

### Approach 1: Pre-order Traversal (Recursive)

```javascript
class TreeNode {
    constructor(val, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

/**
 * Serialize binary tree to string
 * Time: O(n) - Visit each node once
 * Space: O(n) - Store all values
 */
function serialize(root) {
    const result = [];
    
    function preorder(node) {
        if (!node) {
            // Mark null nodes explicitly
            result.push('null');
            return;
        }
        
        // Add current value
        result.push(node.val.toString());
        
        // Recursively serialize left and right
        preorder(node.left);
        preorder(node.right);
    }
    
    preorder(root);
    return result.join(',');
}

/**
 * Deserialize string to binary tree
 * Time: O(n) - Process each value once
 * Space: O(n) - Build tree with n nodes
 */
function deserialize(data) {
    const values = data.split(',');
    let index = 0;
    
    function buildTree() {
        // Check if we've processed all values
        if (index >= values.length) return null;
        
        // Get current value and move pointer
        const val = values[index];
        index++;
        
        // Handle null nodes
        if (val === 'null') return null;
        
        // Create node and recursively build children
        const node = new TreeNode(parseInt(val));
        node.left = buildTree();   // Build left subtree
        node.right = buildTree();  // Build right subtree
        
        return node;
    }
    
    return buildTree();
}

// Test the functions
const tree = new TreeNode(1,
    new TreeNode(2),
    new TreeNode(3,
        new TreeNode(4),
        new TreeNode(5)
    )
);

const serialized = serialize(tree);
console.log("Serialized:", serialized);
// Output: "1,2,null,null,3,4,null,null,5,null,null"

const deserialized = deserialize(serialized);
console.log("Deserialized root:", deserialized.val); // 1
console.log("Left child:", deserialized.left.val);   // 2
console.log("Right child:", deserialized.right.val);  // 3
```

### Approach 2: Level-order Traversal (BFS)

```javascript
/**
 * Serialize using BFS (level-order)
 * More intuitive for visualization
 */
function serializeBFS(root) {
    if (!root) return '';
    
    const result = [];
    const queue = [root];
    
    while (queue.length > 0) {
        const node = queue.shift();
        
        if (node) {
            result.push(node.val);
            // Add children even if null (preserves structure)
            queue.push(node.left);
            queue.push(node.right);
        } else {
            result.push('null');
        }
    }
    
    // Remove trailing nulls for efficiency
    while (result[result.length - 1] === 'null') {
        result.pop();
    }
    
    return result.join(',');
}

/**
 * Deserialize from BFS format
 */
function deserializeBFS(data) {
    if (!data) return null;
    
    const values = data.split(',');
    const root = new TreeNode(parseInt(values[0]));
    const queue = [root];
    let index = 1;
    
    while (queue.length > 0 && index < values.length) {
        const node = queue.shift();
        
        // Process left child
        if (index < values.length && values[index] !== 'null') {
            node.left = new TreeNode(parseInt(values[index]));
            queue.push(node.left);
        }
        index++;
        
        // Process right child
        if (index < values.length && values[index] !== 'null') {
            node.right = new TreeNode(parseInt(values[index]));
            queue.push(node.right);
        }
        index++;
    }
    
    return root;
}
```

## 🎨 Step-by-Step Walkthrough

### Serialization Process (Pre-order)
```
Tree:     1
         / \
        2   3
           / \
          4   5

index = 0, values = []

Process node 1:
- Add "1" → values = ["1"]
- Process left child (2)

Process node 2:
- Add "2" → values = ["1", "2"]
- Process left child (null)
- Add "null" → values = ["1", "2", "null"]
- Process right child (null)
- Add "null" → values = ["1", "2", "null", "null"]

Back to node 1, process right child (3):
- Add "3" → values = ["1", "2", "null", "null", "3"]
- Process left child (4)

Process node 4:
- Add "4" → values = ["1", "2", "null", "null", "3", "4"]
- Add "null" for left → ["1", "2", "null", "null", "3", "4", "null"]
- Add "null" for right → ["1", "2", "null", "null", "3", "4", "null", "null"]

Back to node 3, process right child (5):
- Add "5" → values = ["1", "2", "null", "null", "3", "4", "null", "null", "5"]
- Add "null" for left → ["1", "2", "null", "null", "3", "4", "null", "null", "5", "null"]
- Add "null" for right → ["1", "2", "null", "null", "3", "4", "null", "null", "5", "null", "null"]

Final: "1,2,null,null,3,4,null,null,5,null,null"
```

### Deserialization Process
```
String: "1,2,null,null,3,4,null,null,5,null,null"
values = ["1", "2", "null", "null", "3", "4", "null", "null", "5", "null", "null"]
index = 0

buildTree():
- values[0] = "1" → Create node(1), index = 1
- Build left: values[1] = "2" → Create node(2), index = 2
  - Build left: values[2] = "null" → return null, index = 3
  - Build right: values[3] = "null" → return null, index = 4
- Build right: values[4] = "3" → Create node(3), index = 5
  - Build left: values[5] = "4" → Create node(4), index = 6
    - Build left: values[6] = "null" → return null, index = 7
    - Build right: values[7] = "null" → return null, index = 8
  - Build right: values[8] = "5" → Create node(5), index = 9
    - Build left: values[9] = "null" → return null, index = 10
    - Build right: values[10] = "null" → return null, index = 11

Tree reconstructed!
```

## ⏱️ Complexity Analysis

Both approaches:
- **Time: O(n)** - Visit/process each node exactly once
- **Space: O(n)** - Store string representation of n nodes

Additional space considerations:
- Pre-order: O(h) recursion stack during processing
- Level-order: O(w) queue space where w is max width

## 🚫 Common Mistakes

1. **Forgetting to Mark Null Nodes**
   ```javascript
   // Wrong: Can't distinguish structure
   serialize: "1,2,3,4,5"
   // Is it:    1        or      1     ?
   //          / \              / \
   //         2   3            2   3
   //        / \                  / \
   //       4   5                4   5
   
   // Right: Include nulls
   serialize: "1,2,null,null,3,4,null,null,5,null,null"
   ```

2. **Index Management in Deserialization**
   ```javascript
   // Wrong: Using local index
   function buildTree(values, index) {
       index++; // This doesn't affect caller's index!
   }
   
   // Right: Use shared index
   let index = 0; // Shared across all recursive calls
   ```

3. **Not Handling Empty Trees**
   ```javascript
   // Always check for null root
   if (!root) return '';
   ```

## 🔄 Alternative Formats

### JSON Format
```javascript
function serializeJSON(root) {
    if (!root) return 'null';
    
    return JSON.stringify({
        val: root.val,
        left: serializeJSON(root.left),
        right: serializeJSON(root.right)
    });
}

function deserializeJSON(data) {
    const obj = JSON.parse(data);
    if (obj === null) return null;
    
    const node = new TreeNode(obj.val);
    node.left = deserializeJSON(obj.left);
    node.right = deserializeJSON(obj.right);
    return node;
}
```

### Parentheses Format
```javascript
// Format: val(left)(right)
// Example: 1(2()())(3(4()())(5()()))
function serializeParen(root) {
    if (!root) return '()';
    return `${root.val}(${serializeParen(root.left)})(${serializeParen(root.right)})`;
}
```

## 🔗 Related Problems

- **Serialize and Deserialize BST** - Can optimize using BST properties
- **Serialize and Deserialize N-ary Tree** - Multiple children per node
- **Encode and Decode Strings** - Similar serialization concept
- **Construct Tree from Preorder and Inorder** - Build tree from traversals

## 💡 Key Takeaways

1. **Mark Structure Explicitly**: Null nodes are important!
2. **Pre-order Works Best**: Root-first makes deserialization natural
3. **Index Management**: Share index across recursive calls
4. **Multiple Approaches**: BFS vs DFS, different string formats

This is a fundamental problem that tests tree understanding and recursive thinking!
