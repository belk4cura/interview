# Clone Graph

## 🎯 What Are We Trying To Solve?

Imagine you need to make an exact copy of a social network - every person (node) and every friendship (edge) must be duplicated, but the copy must be completely independent. It's like photocopying a map where all the roads stay connected the same way, but it's a completely new map you can modify without affecting the original.

## 📝 The Problem

Given a reference to a node in a connected undirected graph, create a deep copy of the entire graph. Each node contains a value and a list of its neighbors.

```
Original Graph:          Cloned Graph:
    1 --- 2                 1' --- 2'
    |     |                 |      |
    |     |                 |      |
    4 --- 3                 4' --- 3'

Note: 1' is a new node with same value as 1
```

## 🧠 How to Think About This

The challenge: When we copy a node, we need to also copy its neighbors. But those neighbors might reference back to nodes we're still creating!

Solution: Use a hash map to track what we've already cloned.

### Visual Process
```
Step 1: Clone node 1
  Map: {1 → 1'}
  Need to clone neighbors [2, 4]

Step 2: Clone node 2
  Map: {1 → 1', 2 → 2'}
  Need to clone neighbors [1, 3]
  Node 1 already cloned! Use 1' from map

Step 3: Clone node 3
  Map: {1 → 1', 2 → 2', 3 → 3'}
  Continue...
```

## 💻 The Code

### Approach 1: DFS with HashMap

```javascript
// Node definition
class Node {
    constructor(val, neighbors = []) {
        this.val = val;
        this.neighbors = neighbors;
    }
}

/**
 * Deep copy graph using DFS
 * Time: O(V + E) where V = vertices, E = edges
 * Space: O(V) for the hash map and recursion stack
 */
function cloneGraph(node) {
    // Handle empty graph
    if (!node) return null;
    
    // Map to store original → clone mappings
    const visited = new Map();
    
    function dfs(node) {
        // If already cloned, return the clone
        if (visited.has(node)) {
            return visited.get(node);
        }
        
        // Create clone of current node
        const clonedNode = new Node(node.val);
        
        // Mark as visited BEFORE processing neighbors
        // This prevents infinite recursion in cycles
        visited.set(node, clonedNode);
        
        // Clone all neighbors
        for (const neighbor of node.neighbors) {
            clonedNode.neighbors.push(dfs(neighbor));
        }
        
        return clonedNode;
    }
    
    return dfs(node);
}

// Test the function
// Create graph: 1 -- 2
//               |    |
//               4 -- 3
const node1 = new Node(1);
const node2 = new Node(2);
const node3 = new Node(3);
const node4 = new Node(4);

node1.neighbors = [node2, node4];
node2.neighbors = [node1, node3];
node3.neighbors = [node2, node4];
node4.neighbors = [node1, node3];

const cloned = cloneGraph(node1);
console.log(cloned.val); // 1
console.log(cloned.neighbors.map(n => n.val)); // [2, 4]
console.log(cloned !== node1); // true (different objects)
```

### Approach 2: BFS with Queue

```javascript
/**
 * Clone graph using BFS
 * Time: O(V + E)
 * Space: O(V)
 */
function cloneGraphBFS(node) {
    if (!node) return null;
    
    // Create clone of starting node
    const clonedNode = new Node(node.val);
    const visited = new Map();
    visited.set(node, clonedNode);
    
    // BFS queue
    const queue = [node];
    
    while (queue.length > 0) {
        const current = queue.shift();
        
        // Process all neighbors
        for (const neighbor of current.neighbors) {
            if (!visited.has(neighbor)) {
                // Clone neighbor and add to queue
                visited.set(neighbor, new Node(neighbor.val));
                queue.push(neighbor);
            }
            
            // Connect clone to cloned neighbor
            visited.get(current).neighbors.push(visited.get(neighbor));
        }
    }
    
    return clonedNode;
}
```

### Approach 3: Two-Pass Method

```javascript
/**
 * Clone in two passes:
 * 1st pass: Create all nodes
 * 2nd pass: Connect neighbors
 */
function cloneGraphTwoPass(node) {
    if (!node) return null;
    
    const visited = new Map();
    const queue = [node];
    
    // First pass: Create all cloned nodes
    visited.set(node, new Node(node.val));
    
    while (queue.length > 0) {
        const current = queue.shift();
        
        for (const neighbor of current.neighbors) {
            if (!visited.has(neighbor)) {
                visited.set(neighbor, new Node(neighbor.val));
                queue.push(neighbor);
            }
        }
    }
    
    // Second pass: Connect all edges
    for (const [original, clone] of visited) {
        for (const neighbor of original.neighbors) {
            clone.neighbors.push(visited.get(neighbor));
        }
    }
    
    return visited.get(node);
}
```

## 🎨 Step-by-Step Walkthrough

Let's trace through DFS approach with this graph:
```
    1 --- 2
    |     |
    4 --- 3
```

```
Initial call: cloneGraph(node1)
visited = {}

DFS(node1):
  - Create clone1 with val=1
  - visited = {node1 → clone1}
  - Process neighbors [node2, node4]
  
  DFS(node2):
    - Create clone2 with val=2
    - visited = {node1 → clone1, node2 → clone2}
    - Process neighbors [node1, node3]
    
    DFS(node1):
      - Already in visited! Return clone1
    
    DFS(node3):
      - Create clone3 with val=3
      - visited = {node1 → clone1, node2 → clone2, node3 → clone3}
      - Process neighbors [node2, node4]
      
      DFS(node2):
        - Already in visited! Return clone2
      
      DFS(node4):
        - Create clone4 with val=4
        - visited = {node1 → clone1, ..., node4 → clone4}
        - Process neighbors [node1, node3]
        
        DFS(node1):
          - Already in visited! Return clone1
        
        DFS(node3):
          - Already in visited! Return clone3
        
        - clone4.neighbors = [clone1, clone3]
        - Return clone4
      
      - clone3.neighbors = [clone2, clone4]
      - Return clone3
    
    - clone2.neighbors = [clone1, clone3]
    - Return clone2
  
  DFS(node4):
    - Already in visited! Return clone4
  
  - clone1.neighbors = [clone2, clone4]
  - Return clone1

Final: Complete cloned graph with all connections!
```

## ⏱️ Complexity Analysis

All approaches:
- **Time: O(V + E)** 
  - Visit each vertex once: O(V)
  - Process each edge twice (undirected): O(E)
- **Space: O(V)**
  - Hash map stores V nodes
  - DFS: Recursion stack up to O(V) in worst case
  - BFS: Queue up to O(V) in worst case

## 🚫 Common Mistakes

1. **Infinite Recursion in Cycles**
   ```javascript
   // Wrong: Not tracking visited nodes
   function wrong(node) {
       const clone = new Node(node.val);
       for (const neighbor of node.neighbors) {
           clone.neighbors.push(wrong(neighbor)); // Infinite loop!
       }
       return clone;
   }
   
   // Right: Track visited nodes
   const visited = new Map();
   ```

2. **Creating Duplicate Clones**
   ```javascript
   // Wrong: Creating new clone every time
   for (const neighbor of node.neighbors) {
       cloned.neighbors.push(new Node(neighbor.val)); // Duplicates!
   }
   
   // Right: Reuse existing clones
   for (const neighbor of node.neighbors) {
       cloned.neighbors.push(visited.get(neighbor));
   }
   ```

3. **Shallow Copy Instead of Deep Copy**
   ```javascript
   // Wrong: Sharing neighbor arrays
   const clone = new Node(node.val);
   clone.neighbors = node.neighbors; // Same array reference!
   
   // Right: Create new connections
   clone.neighbors = [];
   for (const neighbor of node.neighbors) {
       clone.neighbors.push(cloneNeighbor);
   }
   ```

## 🔄 Variations

### Clone Directed Graph
```javascript
function cloneDirectedGraph(node) {
    // Same algorithm works!
    // Edges are one-way but approach doesn't change
    if (!node) return null;
    
    const visited = new Map();
    
    function dfs(node) {
        if (visited.has(node)) return visited.get(node);
        
        const clone = new Node(node.val);
        visited.set(node, clone);
        
        // For directed graph, these are "outgoing" edges
        for (const neighbor of node.neighbors) {
            clone.neighbors.push(dfs(neighbor));
        }
        
        return clone;
    }
    
    return dfs(node);
}
```

### Clone Weighted Graph
```javascript
class WeightedNode {
    constructor(val) {
        this.val = val;
        this.edges = []; // Array of {node, weight}
    }
}

function cloneWeightedGraph(node) {
    if (!node) return null;
    
    const visited = new Map();
    
    function dfs(node) {
        if (visited.has(node)) return visited.get(node);
        
        const clone = new WeightedNode(node.val);
        visited.set(node, clone);
        
        for (const edge of node.edges) {
            clone.edges.push({
                node: dfs(edge.node),
                weight: edge.weight
            });
        }
        
        return clone;
    }
    
    return dfs(node);
}
```

## 🔗 Related Problems

- **Copy List with Random Pointer** - Similar concept with linked list
- **Clone Binary Tree** - Simpler version (no cycles)
- **Number of Islands** - Graph traversal problem
- **Course Schedule** - Graph cycle detection

## 💡 Key Takeaways

1. **HashMap is Essential**: Track original → clone mappings
2. **Mark Visited Early**: Prevent infinite recursion in cycles
3. **DFS vs BFS**: Both work, DFS is often simpler
4. **Deep vs Shallow**: Ensure all references are to cloned nodes

This pattern of using a visited map appears in many graph problems!
