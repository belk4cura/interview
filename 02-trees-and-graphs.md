# Trees and Graphs

## Plain English Explanation
A **tree** is like a family tree or company org chart - it starts with one person/CEO at the top (root) and branches out to children/employees below. Each person has exactly one parent (except the root), and there are no loops - you can't be your own grandparent!

A **graph** is like a social network or road map - people/cities (nodes) are connected by friendships/roads (edges). Unlike trees, graphs can have cycles (you can drive in a circle), and nodes can have multiple paths between them.

## Visual Representation
```
Binary Tree:                Graph:                    BST (Binary Search Tree):
       1                    A---B                            10
      / \                   |\ /|                           /  \
     2   3                  | X |                          5    15
    / \   \                 |/ \|                         / \    \
   4   5   6                C---D                        3   7    20
                                                              
Tree: Hierarchical         Graph: Network            BST: Ordered Tree
No cycles                  May have cycles           Left < Parent < Right
One path between nodes     Multiple paths OK         Enables fast search
```

## Overview
Trees and graphs are hierarchical and network data structures fundamental to computer science. They model relationships, hierarchies, and connections found in real-world systems.

### Key Concepts
- **Binary Trees**: Hierarchical structures with at most two children per node
- **Binary Search Trees (BST)**: Ordered trees enabling O(log n) operations
- **Graphs**: Networks of nodes (vertices) connected by edges
- **AWS Applications**: VPC networking, Lambda dependency trees, IAM policy hierarchies

### When to Use What?
- **Use Trees when**: Hierarchical data, need parent-child relationships, no cycles allowed
- **Use BST when**: Need sorted data with fast search/insert/delete
- **Use Graphs when**: Network relationships, multiple paths between nodes, cycles are meaningful
- **Avoid Trees when**: Data has many-to-many relationships
- **Avoid Graphs when**: Simple hierarchical data (trees are simpler)

## Traversal Visualizations

### Tree Traversal Order Examples
```
Given Tree:        
       1
      / \
     2   3
    / \
   4   5

Pre-order (Root → Left → Right):  1 → 2 → 4 → 5 → 3
In-order (Left → Root → Right):   4 → 2 → 5 → 1 → 3
Post-order (Left → Right → Root): 4 → 5 → 2 → 3 → 1
Level-order (Level by level):     1 → 2 → 3 → 4 → 5

DFS vs BFS on Graph:
    A --- B
    |     |
    C --- D
    
DFS from A: A → B → D → C (goes deep first)
BFS from A: A → B → C → D (explores neighbors first)
```

## Binary Trees

### Basic Binary Tree Implementation

```javascript
class TreeNode {
    constructor(value) {
        this.value = value;
        this.left = null;   // Reference to left child
        this.right = null;  // Reference to right child
    }
}

class BinaryTree {
    constructor() {
        this.root = null;  // Start with empty tree
    }
    
    // ============= PRE-ORDER TRAVERSAL =============
    // Time: O(n) - WHY?
    // We visit every node exactly once: n nodes × 1 visit = O(n)
    // Space: O(h) where h is height - WHY?
    // Recursion creates call stack of maximum depth = tree height
    // Worst case: O(n) for skewed tree (height = n)
    // Best case: O(log n) for balanced tree (height = log n)
    preOrderTraversal(node = this.root, result = []) {
        // Base case: reached null (child of leaf node)
        if (!node) return result;
        
        // Pre-order: Root → Left → Right
        // Process parent BEFORE children (that's why "pre")
        result.push(node.value);
        
        // Recursive call adds frame to call stack
        // Left subtree fully explored before right
        this.preOrderTraversal(node.left, result);
        
        // Only reached after ALL left subtree is done
        this.preOrderTraversal(node.right, result);
        
        return result;
    }
    
    // ============= IN-ORDER TRAVERSAL =============
    // Time: O(n) - same reason as pre-order
    // Space: O(h) - same stack depth as pre-order
    // Special property: Gives sorted order for BST!
    inOrderTraversal(node = this.root, result = []) {
        if (!node) return result;
        
        // In-order: Left → Root → Right
        // Process left subtree COMPLETELY first
        this.inOrderTraversal(node.left, result);
        
        // Process current node AFTER left, BEFORE right
        // This is why BST in-order gives sorted sequence
        result.push(node.value);
        
        // Finally process right subtree
        this.inOrderTraversal(node.right, result);
        
        return result;
    }
    
    // ============= POST-ORDER TRAVERSAL =============
    // Time: O(n) - still visiting each node once
    // Space: O(h) - same recursion depth
    // Use case: Delete tree, calculate size, etc.
    postOrderTraversal(node = this.root, result = []) {
        if (!node) return result;
        
        // Post-order: Left → Right → Root
        // Process BOTH children before parent
        this.postOrderTraversal(node.left, result);
        this.postOrderTraversal(node.right, result);
        
        // Parent processed LAST (that's why "post")
        // Useful when you need children info first
        result.push(node.value);
        
        return result;
    }
    
    // ============= LEVEL-ORDER TRAVERSAL (BFS) =============
    // Time: O(n) - visits every node once
    // Space: O(w) where w is max width - WHY?
    // Queue holds at most one complete level
    // Worst case: O(n/2) for complete tree (last level)
    // This is why BFS uses more space than DFS for trees!
    levelOrderTraversal() {
        if (!this.root) return [];
        
        const result = [];
        const queue = [this.root];  // Queue for BFS
        
        // Process nodes level by level
        while (queue.length > 0) {
            // Process all nodes at current level
            const levelSize = queue.length;  // Snapshot current level size
            const currentLevel = [];
            
            // Process exactly levelSize nodes (current level)
            for (let i = 0; i < levelSize; i++) {
                // FIFO: First In First Out (that's why it's BFS)
                const node = queue.shift();  // O(n) operation! (array shift)
                currentLevel.push(node.value);
                
                // Add next level nodes to queue
                // Left child queued before right (left-to-right order)
                if (node.left) queue.push(node.left);
                if (node.right) queue.push(node.right);
            }
            
            result.push(currentLevel);
        }
        
        return result;
    }
    
    // ============= CALCULATE HEIGHT =============
    // Time: O(n) - WHY?
    // Must visit every node to find deepest leaf
    // Can't skip subtrees - might miss deeper branch
    // Space: O(h) - recursion stack depth
    height(node = this.root) {
        // Height of null is -1 (convention)
        // Height of leaf is 0
        if (!node) return -1;
        
        // Recursively calculate height of subtrees
        const leftHeight = this.height(node.left);
        const rightHeight = this.height(node.right);
        
        // Height = max(left, right) + 1
        // +1 accounts for edge to child
        return Math.max(leftHeight, rightHeight) + 1;
    }
}

// Usage
const tree = new BinaryTree();
tree.root = new TreeNode(1);
tree.root.left = new TreeNode(2);
tree.root.right = new TreeNode(3);
tree.root.left.left = new TreeNode(4);
tree.root.left.right = new TreeNode(5);

console.log(tree.inOrderTraversal());    // [4, 2, 5, 1, 3]
console.log(tree.levelOrderTraversal()); // [[1], [2, 3], [4, 5]]
```

## Binary Search Trees (BST)

### BST Implementation with All Operations

```javascript
class BSTNode {
    constructor(value) {
        this.value = value;
        this.left = null;
        this.right = null;
    }
}

class BinarySearchTree {
    constructor() {
        this.root = null;
    }
    
    // O(log n) average, O(n) worst case
    insert(value) {
        const newNode = new BSTNode(value);
        
        if (!this.root) {
            this.root = newNode;
            return this;
        }
        
        let current = this.root;
        while (true) {
            if (value === current.value) return undefined; // No duplicates
            
            if (value < current.value) {
                if (!current.left) {
                    current.left = newNode;
                    return this;
                }
                current = current.left;
            } else {
                if (!current.right) {
                    current.right = newNode;
                    return this;
                }
                current = current.right;
            }
        }
    }
    
    // O(log n) average, O(n) worst case
    search(value) {
        let current = this.root;
        
        while (current) {
            if (value === current.value) return current;
            if (value < current.value) {
                current = current.left;
            } else {
                current = current.right;
            }
        }
        
        return null;
    }
    
    // O(log n) average, O(n) worst case
    delete(value) {
        this.root = this.deleteNode(this.root, value);
    }
    
    deleteNode(node, value) {
        if (!node) return null;
        
        if (value < node.value) {
            node.left = this.deleteNode(node.left, value);
        } else if (value > node.value) {
            node.right = this.deleteNode(node.right, value);
        } else {
            // Node to be deleted found
            
            // Case 1: No children (leaf node)
            if (!node.left && !node.right) {
                return null;
            }
            
            // Case 2: One child
            if (!node.left) return node.right;
            if (!node.right) return node.left;
            
            // Case 3: Two children
            // Find inorder successor (min value in right subtree)
            let minNode = this.findMin(node.right);
            node.value = minNode.value;
            node.right = this.deleteNode(node.right, minNode.value);
        }
        
        return node;
    }
    
    // O(log n) average, O(n) worst case
    findMin(node = this.root) {
        while (node.left) {
            node = node.left;
        }
        return node;
    }
    
    // O(log n) average, O(n) worst case
    findMax(node = this.root) {
        while (node.right) {
            node = node.right;
        }
        return node;
    }
    
    // O(n) - must visit all nodes
    isValidBST(node = this.root, min = -Infinity, max = Infinity) {
        if (!node) return true;
        
        if (node.value <= min || node.value >= max) {
            return false;
        }
        
        return this.isValidBST(node.left, min, node.value) &&
               this.isValidBST(node.right, node.value, max);
    }
    
    // O(log n) average
    findLCA(node, p, q) {
        if (!node) return null;
        
        // If both values are less than node, LCA is in left subtree
        if (p < node.value && q < node.value) {
            return this.findLCA(node.left, p, q);
        }
        
        // If both values are greater than node, LCA is in right subtree
        if (p > node.value && q > node.value) {
            return this.findLCA(node.right, p, q);
        }
        
        // Otherwise, this node is the LCA
        return node;
    }
}

// Usage
const bst = new BinarySearchTree();
[15, 10, 20, 8, 12, 17, 25].forEach(val => bst.insert(val));
console.log(bst.search(12));  // Returns node with value 12
console.log(bst.isValidBST()); // true
```

### Balanced Trees - AVL Tree

```javascript
class AVLNode {
    constructor(value) {
        this.value = value;
        this.left = null;
        this.right = null;
        this.height = 0;
    }
}

class AVLTree {
    constructor() {
        this.root = null;
    }
    
    getHeight(node) {
        return node ? node.height : -1;
    }
    
    getBalance(node) {
        return node ? this.getHeight(node.left) - this.getHeight(node.right) : 0;
    }
    
    updateHeight(node) {
        if (node) {
            node.height = Math.max(this.getHeight(node.left), this.getHeight(node.right)) + 1;
        }
    }
    
    // O(1) rotation operations
    rotateRight(y) {
        const x = y.left;
        const T2 = x.right;
        
        x.right = y;
        y.left = T2;
        
        this.updateHeight(y);
        this.updateHeight(x);
        
        return x;
    }
    
    rotateLeft(x) {
        const y = x.right;
        const T2 = y.left;
        
        y.left = x;
        x.right = T2;
        
        this.updateHeight(x);
        this.updateHeight(y);
        
        return y;
    }
    
    // O(log n) guaranteed
    insert(value) {
        this.root = this.insertNode(this.root, value);
    }
    
    insertNode(node, value) {
        // Standard BST insertion
        if (!node) return new AVLNode(value);
        
        if (value < node.value) {
            node.left = this.insertNode(node.left, value);
        } else if (value > node.value) {
            node.right = this.insertNode(node.right, value);
        } else {
            return node; // Duplicate values not allowed
        }
        
        // Update height
        this.updateHeight(node);
        
        // Get balance factor
        const balance = this.getBalance(node);
        
        // Left-Left case
        if (balance > 1 && value < node.left.value) {
            return this.rotateRight(node);
        }
        
        // Right-Right case
        if (balance < -1 && value > node.right.value) {
            return this.rotateLeft(node);
        }
        
        // Left-Right case
        if (balance > 1 && value > node.left.value) {
            node.left = this.rotateLeft(node.left);
            return this.rotateRight(node);
        }
        
        // Right-Left case
        if (balance < -1 && value < node.right.value) {
            node.right = this.rotateRight(node.right);
            return this.rotateLeft(node);
        }
        
        return node;
    }
}
```

## Graphs

### Graph Representations

```javascript
// 1. Adjacency List (Most Common)
class GraphAdjList {
    constructor(directed = false) {
        this.adjacencyList = new Map();
        this.directed = directed;
    }
    
    // O(1)
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    // O(1) for adjacency list
    addEdge(vertex1, vertex2, weight = 1) {
        this.addVertex(vertex1);
        this.addVertex(vertex2);
        
        this.adjacencyList.get(vertex1).push({ node: vertex2, weight });
        
        if (!this.directed) {
            this.adjacencyList.get(vertex2).push({ node: vertex1, weight });
        }
    }
    
    // O(E) where E is number of edges from vertex
    removeEdge(vertex1, vertex2) {
        this.adjacencyList.set(
            vertex1,
            this.adjacencyList.get(vertex1).filter(v => v.node !== vertex2)
        );
        
        if (!this.directed) {
            this.adjacencyList.set(
                vertex2,
                this.adjacencyList.get(vertex2).filter(v => v.node !== vertex1)
            );
        }
    }
}

// 2. Adjacency Matrix
class GraphAdjMatrix {
    constructor(numVertices) {
        this.numVertices = numVertices;
        this.matrix = Array(numVertices).fill(null)
            .map(() => Array(numVertices).fill(0));
    }
    
    // O(1)
    addEdge(vertex1, vertex2, weight = 1) {
        this.matrix[vertex1][vertex2] = weight;
        this.matrix[vertex2][vertex1] = weight; // For undirected
    }
    
    // O(1)
    hasEdge(vertex1, vertex2) {
        return this.matrix[vertex1][vertex2] !== 0;
    }
}

// 3. Edge List
class GraphEdgeList {
    constructor() {
        this.edges = [];
        this.vertices = new Set();
    }
    
    // O(1)
    addEdge(vertex1, vertex2, weight = 1) {
        this.edges.push({ from: vertex1, to: vertex2, weight });
        this.vertices.add(vertex1);
        this.vertices.add(vertex2);
    }
}
```

### Graph Traversal - DFS vs BFS

```javascript
class Graph {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(v1, v2) {
        this.addVertex(v1);
        this.addVertex(v2);
        this.adjacencyList.get(v1).push(v2);
        this.adjacencyList.get(v2).push(v1);
    }
    
    // DFS - Depth First Search
    // Time: O(V + E), Space: O(V)
    dfsRecursive(start, visited = new Set()) {
        const result = [];
        
        const dfs = (vertex) => {
            if (!vertex) return;
            
            visited.add(vertex);
            result.push(vertex);
            
            this.adjacencyList.get(vertex).forEach(neighbor => {
                if (!visited.has(neighbor)) {
                    dfs(neighbor);
                }
            });
        };
        
        dfs(start);
        return result;
    }
    
    // DFS Iterative using Stack
    // Time: O(V + E), Space: O(V)
    dfsIterative(start) {
        const stack = [start];
        const visited = new Set();
        const result = [];
        
        visited.add(start);
        
        while (stack.length) {
            const vertex = stack.pop();
            result.push(vertex);
            
            this.adjacencyList.get(vertex).forEach(neighbor => {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    stack.push(neighbor);
                }
            });
        }
        
        return result;
    }
    
    // BFS - Breadth First Search
    // Time: O(V + E), Space: O(V)
    bfs(start) {
        const queue = [start];
        const visited = new Set();
        const result = [];
        
        visited.add(start);
        
        while (queue.length) {
            const vertex = queue.shift();
            result.push(vertex);
            
            this.adjacencyList.get(vertex).forEach(neighbor => {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    queue.push(neighbor);
                }
            });
        }
        
        return result;
    }
    
    // Find shortest path using BFS
    // Time: O(V + E), Space: O(V)
    shortestPath(start, end) {
        const queue = [[start]];
        const visited = new Set([start]);
        
        while (queue.length) {
            const path = queue.shift();
            const vertex = path[path.length - 1];
            
            if (vertex === end) {
                return path;
            }
            
            for (const neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    queue.push([...path, neighbor]);
                }
            }
        }
        
        return null; // No path found
    }
    
    // Detect cycle in undirected graph
    // Time: O(V + E), Space: O(V)
    hasCycle() {
        const visited = new Set();
        
        const dfs = (vertex, parent) => {
            visited.add(vertex);
            
            for (const neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    if (dfs(neighbor, vertex)) return true;
                } else if (neighbor !== parent) {
                    return true; // Cycle detected
                }
            }
            
            return false;
        };
        
        for (const vertex of this.adjacencyList.keys()) {
            if (!visited.has(vertex)) {
                if (dfs(vertex, null)) return true;
            }
        }
        
        return false;
    }
}

// Usage
const graph = new Graph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph.addVertex(v));
graph.addEdge('A', 'B');
graph.addEdge('A', 'C');
graph.addEdge('B', 'D');
graph.addEdge('C', 'E');
graph.addEdge('D', 'E');
graph.addEdge('D', 'F');
graph.addEdge('E', 'F');

console.log(graph.dfsRecursive('A')); // ['A', 'B', 'D', 'E', 'C', 'F']
console.log(graph.bfs('A'));          // ['A', 'B', 'C', 'D', 'E', 'F']
```

## DFS vs BFS Comparison

| Aspect | DFS | BFS | Use DFS When | Use BFS When |
|--------|-----|-----|--------------|--------------|
| Data Structure | Stack (or recursion) | Queue | - | - |
| Memory | O(h) where h is max depth | O(w) where w is max width | Limited memory, deep graphs | Wide graphs |
| Complete? | No (can get stuck in infinite) | Yes | - | Need to find all solutions |
| Optimal? | No | Yes (for unweighted) | Any path is ok | Need shortest path |
| Implementation | Simpler (recursion) | Iterative | Quick implementation | - |
| Use Cases | Topological sort, maze solving | Shortest path, level order | Path exists check | Distance/levels matter |

## AWS Applications

### 1. VPC Network Topology
```javascript
// VPC routing tables as directed graph
class VPCNetwork {
    constructor() {
        this.routingTable = new Map();
    }
    
    addSubnet(subnetId, cidr) {
        if (!this.routingTable.has(subnetId)) {
            this.routingTable.set(subnetId, {
                cidr,
                routes: []
            });
        }
    }
    
    addRoute(fromSubnet, toSubnet, via = 'direct') {
        const subnet = this.routingTable.get(fromSubnet);
        subnet.routes.push({
            destination: toSubnet,
            via, // 'nat', 'igw', 'peering', 'direct'
            priority: this.calculatePriority(via)
        });
    }
    
    // Find path between subnets using BFS
    findNetworkPath(source, destination) {
        const queue = [[source]];
        const visited = new Set([source]);
        
        while (queue.length) {
            const path = queue.shift();
            const current = path[path.length - 1];
            
            if (current === destination) {
                return this.enrichPathWithDetails(path);
            }
            
            const subnet = this.routingTable.get(current);
            if (subnet) {
                for (const route of subnet.routes) {
                    if (!visited.has(route.destination)) {
                        visited.add(route.destination);
                        queue.push([...path, route.destination]);
                    }
                }
            }
        }
        
        return null; // No route available
    }
    
    calculatePriority(via) {
        const priorities = {
            'direct': 1,
            'peering': 2,
            'nat': 3,
            'igw': 4
        };
        return priorities[via] || 999;
    }
    
    enrichPathWithDetails(path) {
        const details = [];
        for (let i = 0; i < path.length - 1; i++) {
            const subnet = this.routingTable.get(path[i]);
            const route = subnet.routes.find(r => r.destination === path[i + 1]);
            details.push({
                from: path[i],
                to: path[i + 1],
                via: route.via,
                cidr: subnet.cidr
            });
        }
        return details;
    }
}
```

### 2. Lambda Dependency Tree
```javascript
// Lambda function dependencies as DAG
class LambdaDependencyGraph {
    constructor() {
        this.functions = new Map();
    }
    
    addFunction(name, config) {
        this.functions.set(name, {
            ...config,
            dependencies: [],
            dependents: []
        });
    }
    
    addDependency(functionName, dependsOn) {
        const func = this.functions.get(functionName);
        const dependency = this.functions.get(dependsOn);
        
        func.dependencies.push(dependsOn);
        dependency.dependents.push(functionName);
    }
    
    // Topological sort for deployment order
    // Time: O(V + E)
    getDeploymentOrder() {
        const inDegree = new Map();
        const queue = [];
        const result = [];
        
        // Calculate in-degrees
        for (const [name, func] of this.functions) {
            inDegree.set(name, func.dependencies.length);
            if (func.dependencies.length === 0) {
                queue.push(name);
            }
        }
        
        // Process queue
        while (queue.length) {
            const current = queue.shift();
            result.push(current);
            
            const func = this.functions.get(current);
            for (const dependent of func.dependents) {
                inDegree.set(dependent, inDegree.get(dependent) - 1);
                if (inDegree.get(dependent) === 0) {
                    queue.push(dependent);
                }
            }
        }
        
        // Check for cycles
        if (result.length !== this.functions.size) {
            throw new Error('Circular dependency detected');
        }
        
        return result;
    }
    
    // Find all functions affected by a change
    findImpactedFunctions(changedFunction) {
        const impacted = new Set();
        const queue = [changedFunction];
        
        while (queue.length) {
            const current = queue.shift();
            const func = this.functions.get(current);
            
            for (const dependent of func.dependents) {
                if (!impacted.has(dependent)) {
                    impacted.add(dependent);
                    queue.push(dependent);
                }
            }
        }
        
        return Array.from(impacted);
    }
}
```

### 3. IAM Policy Evaluation Tree
```javascript
// IAM policy evaluation as tree traversal
class IAMPolicyTree {
    constructor() {
        this.root = {
            effect: 'Deny', // Default deny
            conditions: [],
            children: new Map()
        };
    }
    
    addPolicy(path, effect, conditions = []) {
        const parts = path.split('/');
        let current = this.root;
        
        for (const part of parts) {
            if (!current.children.has(part)) {
                current.children.set(part, {
                    effect: 'Inherit',
                    conditions: [],
                    children: new Map()
                });
            }
            current = current.children.get(part);
        }
        
        current.effect = effect;
        current.conditions = conditions;
    }
    
    // Evaluate permission using DFS
    evaluatePermission(resource, principal, action) {
        const path = resource.split('/');
        let current = this.root;
        let finalEffect = current.effect;
        
        for (const part of path) {
            if (current.children.has(part)) {
                current = current.children.get(part);
                
                // Check conditions
                if (this.checkConditions(current.conditions, principal, action)) {
                    if (current.effect !== 'Inherit') {
                        finalEffect = current.effect;
                    }
                }
            } else if (current.children.has('*')) {
                // Wildcard match
                current = current.children.get('*');
                if (this.checkConditions(current.conditions, principal, action)) {
                    if (current.effect !== 'Inherit') {
                        finalEffect = current.effect;
                    }
                }
            } else {
                break;
            }
        }
        
        // Explicit deny always wins
        return finalEffect === 'Allow';
    }
    
    checkConditions(conditions, principal, action) {
        return conditions.every(condition => 
            this.evaluateCondition(condition, principal, action)
        );
    }
    
    evaluateCondition(condition, principal, action) {
        // Simplified condition evaluation
        switch (condition.type) {
            case 'StringEquals':
                return condition.key === condition.value;
            case 'IpAddress':
                return this.matchIpAddress(principal.ip, condition.value);
            case 'DateGreaterThan':
                return new Date() > new Date(condition.value);
            default:
                return true;
        }
    }
    
    matchIpAddress(ip, cidr) {
        // Simplified IP matching logic
        return true; // Actual implementation would check CIDR
    }
}
```

## Practice Problems

### Problem 1: Binary Tree Maximum Path Sum
```javascript
/**
 * Find maximum path sum in binary tree
 * Time: O(n), Space: O(h) where h is height
 */
function maxPathSum(root) {
    let maxSum = -Infinity;
    
    function maxGain(node) {
        if (!node) return 0;
        
        // Recursively get max gain from left and right subtrees
        const leftGain = Math.max(maxGain(node.left), 0);
        const rightGain = Math.max(maxGain(node.right), 0);
        
        // Current path sum including this node
        const currentMax = node.value + leftGain + rightGain;
        
        // Update global maximum
        maxSum = Math.max(maxSum, currentMax);
        
        // Return max gain if we continue path through this node
        return node.value + Math.max(leftGain, rightGain);
    }
    
    maxGain(root);
    return maxSum;
}
```

### Problem 2: Serialize and Deserialize Binary Tree
```javascript
/**
 * Serialize binary tree to string
 * Time: O(n), Space: O(n)
 */
function serialize(root) {
    const result = [];
    
    function preorder(node) {
        if (!node) {
            result.push('null');
            return;
        }
        result.push(node.value);
        preorder(node.left);
        preorder(node.right);
    }
    
    preorder(root);
    return result.join(',');
}

/**
 * Deserialize string to binary tree
 * Time: O(n), Space: O(n)
 */
function deserialize(data) {
    const values = data.split(',');
    let index = 0;
    
    function buildTree() {
        if (values[index] === 'null') {
            index++;
            return null;
        }
        
        const node = new TreeNode(parseInt(values[index]));
        index++;
        node.left = buildTree();
        node.right = buildTree();
        
        return node;
    }
    
    return buildTree();
}
```

### Problem 3: Clone Graph
```javascript
/**
 * Deep copy of graph
 * Time: O(V + E), Space: O(V)
 */
function cloneGraph(node) {
    if (!node) return null;
    
    const visited = new Map();
    
    function clone(node) {
        if (visited.has(node)) {
            return visited.get(node);
        }
        
        const clonedNode = { value: node.value, neighbors: [] };
        visited.set(node, clonedNode);
        
        for (const neighbor of node.neighbors) {
            clonedNode.neighbors.push(clone(neighbor));
        }
        
        return clonedNode;
    }
    
    return clone(node);
}
```

## Interview Tips

### Common Questions
1. **"When would you use DFS vs BFS?"**
   - DFS: Detecting cycles, topological sort, pathfinding in mazes
   - BFS: Shortest path, level-order traversal, finding connected components

2. **"How do you detect a cycle in a directed graph?"**
   - Use DFS with three colors: white (unvisited), gray (visiting), black (visited)
   - Cycle exists if we encounter a gray node

3. **"Explain tree balancing and why it matters"**
   - Prevents O(n) degradation in BST operations
   - AVL ensures O(log n) through rotations
   - Red-Black trees used in many libraries

### Red Flags to Avoid
- ❌ Forgetting to check for cycles in graph algorithms
- ❌ Not handling disconnected components in graphs
- ❌ Using recursion without considering stack overflow
- ❌ Forgetting null checks in tree traversals

### AWS-Specific Follow-ups
- "How would you model AWS service dependencies?"
- "Design a system to detect circular dependencies in CloudFormation"
- "Implement VPC peering connection validation"
- "How does Route 53 use tree structures for DNS resolution?"

## Quiz Questions (With Answers)

### Q1: What's the difference between a tree and a graph?

**Answer:** A tree is a special type of graph with specific constraints:
1. **Trees are acyclic**: No cycles allowed (can't return to a node via edges)
2. **Trees are connected**: There's exactly one path between any two nodes
3. **Trees have n-1 edges**: For n nodes, always n-1 edges
4. **Trees have a root**: One designated starting node (except in forest)
5. **Directed hierarchy**: Clear parent-child relationships

Graphs have none of these restrictions - they can have cycles, be disconnected, have any number of edges, and no hierarchical structure.

### Q2: Why does BST search degrade to O(n) and how do we prevent it?

**Answer:** BST search degrades to O(n) when the tree becomes unbalanced (skewed):

```
Balanced BST:     Skewed BST (essentially a linked list):
      10                10
     /  \                 \
    5    15                15
   / \    \                 \
  3   7    20                20
                              \
                              25
O(log n)           O(n) search time
```

**Prevention methods:**
1. **Self-balancing trees**: AVL (strict balance), Red-Black (relaxed balance)
2. **Rotations**: Rebalance through left/right rotations
3. **B-trees**: Multiple values per node (used in databases)
4. **Random insertion order**: Reduces chance of worst case

### Q3: When should you use DFS vs BFS in real applications?

**Answer:** 
**Use DFS when:**
- Finding ANY path (not shortest): DFS finds a path quickly
- Detecting cycles: Easy with recursion stack
- Topological sorting: Natural fit for dependencies
- Memory is limited: O(h) vs O(w) space
- Maze solving: Explore one path fully before backtracking

**Use BFS when:**
- Finding SHORTEST path: BFS guarantees shortest in unweighted graphs
- Level-order processing: Web crawlers, social network analysis
- Finding all nodes at distance k: Friend suggestions
- Checking if graph is bipartite: Alternate coloring by levels
- State space exploration: Game AI, puzzle solving

**Real AWS Example:** Route 53 health checks use BFS to find the shortest healthy path to your resources.

### Q4: How do you detect a cycle in a directed graph?

**Answer:** Use DFS with three colors (states):

```javascript
function hasCycleDirected(graph) {
    const WHITE = 0; // Unvisited
    const GRAY = 1;  // Currently visiting (in recursion stack)
    const BLACK = 2; // Completely visited
    
    const colors = new Map();
    
    // Initialize all nodes as white
    for (const node of graph.nodes) {
        colors.set(node, WHITE);
    }
    
    function dfs(node) {
        colors.set(node, GRAY); // Mark as being processed
        
        for (const neighbor of graph.getNeighbors(node)) {
            if (colors.get(neighbor) === GRAY) {
                return true; // Found back edge (cycle)!
            }
            if (colors.get(neighbor) === WHITE) {
                if (dfs(neighbor)) return true;
            }
        }
        
        colors.set(node, BLACK); // Mark as completely processed
        return false;
    }
    
    // Check all components
    for (const node of graph.nodes) {
        if (colors.get(node) === WHITE) {
            if (dfs(node)) return true;
        }
    }
    
    return false;
}
```

**Why it works:** A cycle exists if we encounter a GRAY node (currently in our recursion path). Time: O(V+E), Space: O(V).

### Q5: What's the difference between Adjacency List vs Matrix representation?

**Answer:**

| Aspect | Adjacency List | Adjacency Matrix |
|--------|---------------|------------------|
| **Space** | O(V + E) | O(V²) |
| **Add Edge** | O(1) | O(1) |
| **Remove Edge** | O(E) worst case | O(1) |
| **Check Edge Exists** | O(E) worst case | O(1) |
| **Get All Neighbors** | O(degree) | O(V) |
| **Best For** | Sparse graphs (E << V²) | Dense graphs (E ≈ V²) |

**When to use:**
- **Adjacency List**: Social networks (sparse), Web graphs, Most real-world graphs
- **Adjacency Matrix**: Complete/near-complete graphs, Quick edge lookups needed, Small graphs where V² memory is acceptable

**AWS Example:** DynamoDB's internal graph of item relationships uses adjacency lists because most items relate to few others (sparse).

## Common Mistakes to Avoid

1. **Infinite Recursion in DFS**
   ```javascript
   // BAD: No visited tracking
   function dfs(node) {
       for (const neighbor of node.neighbors) {
           dfs(neighbor); // Infinite loop if cycle!
       }
   }
   
   // GOOD: Track visited
   function dfs(node, visited = new Set()) {
       if (visited.has(node)) return;
       visited.add(node);
       // ... process
   }
   ```

2. **Wrong Tree Traversal for Use Case**
   - Don't use in-order for non-BST (meaningless order)
   - Don't use recursion for very deep trees (stack overflow)

3. **Not Handling Disconnected Components**
   ```javascript
   // BAD: Only processes one component
   const visited = bfs(startNode);
   
   // GOOD: Process all components
   for (const node of graph.nodes) {
       if (!visited.has(node)) {
           bfs(node, visited);
       }
   }
   ```

4. **BST Insertion Without Balancing**
   - Always consider if tree might become skewed
   - Use self-balancing tree for production code

## Key Takeaways
1. **Trees** are special cases of graphs (acyclic, connected)
2. **BST operations** depend on balance for O(log n) performance
3. **Graph representation** choice affects algorithm performance
4. **DFS uses stack**, **BFS uses queue** - fundamental difference
5. **AWS services** extensively use graphs for networking and dependencies
6. **Always handle cycles** in graph algorithms unless guaranteed tree
7. **Consider memory layout** - trees often have better cache locality than graphs

---

*Previous: [← Arrays and Lists](./01-arrays-and-lists.md)*
*Next: [Hash Maps and Sets →](./03-hash-maps-and-sets.md)*
