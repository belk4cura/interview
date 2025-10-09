# Graph Algorithms

## Overview
Graph algorithms are essential for solving network, routing, and relationship problems. They form the backbone of many AWS services and distributed systems, from VPC routing to social network analysis.

### Key Concepts
- **Traversal**: BFS and DFS strategies
- **Shortest Paths**: Dijkstra, Bellman-Ford, Floyd-Warshall
- **Minimum Spanning Trees**: Kruskal, Prim
- **AWS Applications**: VPC routing, network optimization, dependency resolution

## Graph Traversal (Review)

### Breadth-First Search (BFS)
```javascript
// ============= BFS - LEVEL BY LEVEL TRAVERSAL =============
// Time: O(V + E) - WHY?
// Visit each vertex once: O(V)
// Check each edge twice (undirected): O(E)
// Total: O(V + E)
// Space: O(V) for visited set and queue
class Graph {
    constructor() {
        // Using Map for O(1) vertex operations
        // Better than object for non-string keys
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        // Only add if doesn't exist (prevent overwriting edges)
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(v1, v2) {
        // Ensure vertices exist before adding edge
        this.addVertex(v1);
        this.addVertex(v2);
        // Undirected graph - edge goes both ways
        this.adjacencyList.get(v1).push(v2);
        this.adjacencyList.get(v2).push(v1);
    }
    
    // ============= STANDARD BFS =============
    bfs(start) {
        const queue = [start];  // FIFO for level-order
        const visited = new Set([start]);  // O(1) lookups
        const result = [];
        
        // Process vertices level by level
        while (queue.length) {
            // WHY shift() not pop()?
            // shift() = dequeue (FIFO) for BFS
            // pop() would make it DFS (LIFO)
            const vertex = queue.shift();
            result.push(vertex);
            
            // Explore all neighbors at current level
            for (const neighbor of this.adjacencyList.get(vertex)) {
                // WHY check visited before queuing?
                // Prevents: 1) Infinite loops in cycles
                //          2) Duplicate processing
                //          3) Exponential time complexity
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);  // Mark immediately
                    queue.push(neighbor);   // Queue for next level
                }
            }
        }
        
        return result;
    }
    
    // ============= BFS WITH LEVEL TRACKING =============
    // Useful for: shortest path length, tree depth, level-order
    bfsLevels(start) {
        const queue = [start];
        const visited = new Set([start]);
        const levels = [];  // 2D array of levels
        
        while (queue.length) {
            // KEY INSIGHT: Process entire level at once
            // Current queue size = number of nodes at this level
            const levelSize = queue.length;
            const currentLevel = [];
            
            // Process exactly levelSize nodes (current level)
            for (let i = 0; i < levelSize; i++) {
                const vertex = queue.shift();
                currentLevel.push(vertex);
                
                // Queue all unvisited neighbors for next level
                for (const neighbor of this.adjacencyList.get(vertex)) {
                    if (!visited.has(neighbor)) {
                        visited.add(neighbor);
                        queue.push(neighbor);  // Will be processed in next iteration
                    }
                }
            }
            
            levels.push(currentLevel);
        }
        
        return levels;  // [[level0], [level1], [level2], ...]
    }
}

// BFS GUARANTEES:
// 1. Shortest path in unweighted graphs
// 2. Visits nodes in increasing distance order
// 3. Each edge examined at most twice (undirected)
```

### Depth-First Search (DFS)
```javascript
// ============= DFS - EXPLORE AS FAR AS POSSIBLE =============
// Time: O(V + E) - same as BFS but different order
// Space: O(V) for visited set + recursion stack
class GraphDFS {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    // ============= RECURSIVE DFS =============
    // More intuitive but uses call stack
    // Space: O(V) worst case for deep recursion
    dfsRecursive(start, visited = new Set()) {
        const result = [];
        
        // Inner recursive function
        const dfs = (vertex) => {
            // Mark as visited IMMEDIATELY
            // WHY? Prevent infinite recursion in cycles
            visited.add(vertex);
            result.push(vertex);
            
            // Recursively visit all unvisited neighbors
            for (const neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    dfs(neighbor);  // Go deeper before going wider
                }
            }
            // Implicit backtracking when all neighbors visited
        };
        
        dfs(start);
        return result;
    }
    
    // ============= ITERATIVE DFS =============
    // Uses explicit stack instead of call stack
    // Better for deep graphs (no stack overflow)
    dfsIterative(start) {
        const stack = [start];  // LIFO for depth-first
        const visited = new Set([start]);
        const result = [];
        
        while (stack.length) {
            // WHY pop() not shift()?
            // pop() = LIFO behavior (stack) for DFS
            // shift() would make it BFS (queue)
            const vertex = stack.pop();
            result.push(vertex);
            
            // Add all unvisited neighbors to stack
            for (const neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    stack.push(neighbor);  // Will be processed next
                }
            }
        }
        
        return result;
    }
    
    // ============= DFS FOR DISCONNECTED GRAPH =============
    // Finds all connected components
    // Time: O(V + E) total, not per component
    dfsAllComponents() {
        const visited = new Set();  // Global visited set
        const components = [];
        
        // Try starting DFS from each unvisited vertex
        for (const vertex of this.adjacencyList.keys()) {
            if (!visited.has(vertex)) {
                // New component found
                // Pass same visited set to track globally
                const component = this.dfsRecursive(vertex, visited);
                components.push(component);
            }
        }
        
        return components;  // Array of arrays (each is a component)
    }
}

// DFS CHARACTERISTICS:
// 1. Goes deep before going wide
// 2. Natural for: backtracking, cycle detection, path finding
// 3. Order depends on adjacency list order
// 4. Can find ANY path (not necessarily shortest)
```

## Shortest Path Algorithms

### Dijkstra's Algorithm
```javascript
// ============= DIJKSTRA'S - SINGLE SOURCE SHORTEST PATH =============
// Time: O((V + E) log V) with binary heap
// Space: O(V) for distances and priority queue
// LIMITATION: Cannot handle negative edge weights
class WeightedGraph {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(v1, v2, weight) {
        this.addVertex(v1);
        this.addVertex(v2);
        // Store as objects with node and weight
        this.adjacencyList.get(v1).push({ node: v2, weight });
        this.adjacencyList.get(v2).push({ node: v1, weight });
    }
    
    // ============= DIJKSTRA'S ALGORITHM =============
    dijkstra(start, end) {
        const distances = {};  // Shortest known distance to each vertex
        const previous = {};   // Previous vertex in optimal path
        const pq = new PriorityQueue();  // Min heap of unvisited vertices
        
        // INITIALIZATION: Set all distances to infinity except start
        for (const vertex of this.adjacencyList.keys()) {
            if (vertex === start) {
                distances[vertex] = 0;
                pq.enqueue(vertex, 0);  // Start has priority 0
            } else {
                distances[vertex] = Infinity;
                pq.enqueue(vertex, Infinity);
            }
            previous[vertex] = null;  // No paths found yet
        }
        
        // MAIN ALGORITHM: Process vertices by minimum distance
        while (!pq.isEmpty()) {
            // Get vertex with minimum distance
            // GREEDY CHOICE: Always pick closest unvisited vertex
            const current = pq.dequeue();
            
            // OPTIMIZATION: Early termination when target reached
            // WHY VALID? Dijkstra guarantees shortest path when dequeued
            if (current === end) {
                // Reconstruct path from end to start
                const path = [];
                let vertex = end;
                // Backtrack using previous pointers
                while (vertex !== null) {
                    path.unshift(vertex);  // Add to front
                    vertex = previous[vertex];
                }
                return { path, distance: distances[end] };
            }
            
            // Skip if unreachable (disconnected component)
            if (distances[current] !== Infinity) {
                // RELAXATION: Check if we can improve neighbor distances
                for (const neighbor of this.adjacencyList.get(current)) {
                    // Calculate distance through current vertex
                    const alt = distances[current] + neighbor.weight;
                    
                    // If shorter path found, update it
                    if (alt < distances[neighbor.node]) {
                        distances[neighbor.node] = alt;
                        previous[neighbor.node] = current;  // Update path
                        // Re-prioritize with new distance
                        pq.enqueue(neighbor.node, alt);
                    }
                }
            }
        }
        
        // No path exists
        return { path: [], distance: Infinity };
    }
}

// ============= SIMPLE PRIORITY QUEUE =============
// Note: For production, use binary heap for O(log n) operations
// This implementation is O(n log n) due to sorting
class PriorityQueue {
    constructor() {
        this.values = [];
    }
    
    enqueue(val, priority) {
        this.values.push({ val, priority });
        this.sort();  // O(n log n) - inefficient but simple
    }
    
    dequeue() {
        return this.values.shift().val;  // O(n) due to shift
    }
    
    sort() {
        // Sort by priority (ascending for min heap)
        this.values.sort((a, b) => a.priority - b.priority);
    }
    
    isEmpty() {
        return this.values.length === 0;
    }
}

// WHY DIJKSTRA WORKS:
// 1. Greedy: Always picks minimum distance vertex
// 2. No negative weights: Once visited, distance is final
// 3. Relaxation: Continuously improves distance estimates
// 4. Optimal substructure: Shortest path contains shortest subpaths
```

### Bellman-Ford Algorithm
```javascript
// ============= BELLMAN-FORD - HANDLES NEGATIVE WEIGHTS =============
// Time: O(VE) - WHY?
// Outer loop: V-1 iterations
// Inner loop: E edges checked each time
// Total: (V-1) × E = O(VE)
// Space: O(V) for distances and previous arrays
// ADVANTAGE: Handles negative weights and detects negative cycles
function bellmanFord(edges, vertices, source) {
    const distances = {};
    const previous = {};
    
    // INITIALIZATION: All distances infinite except source
    for (const vertex of vertices) {
        distances[vertex] = vertex === source ? 0 : Infinity;
        previous[vertex] = null;
    }
    
    // RELAXATION PHASE: Relax all edges V-1 times
    // WHY V-1 TIMES?
    // Shortest path has at most V-1 edges (no cycles)
    // Each iteration guarantees correct distance for paths of length i
    // After V-1 iterations, all shortest paths are found
    for (let i = 0; i < vertices.length - 1; i++) {
        // Check every edge for possible improvement
        for (const edge of edges) {
            const { from, to, weight } = edge;
            
            // Skip if source vertex is unreachable
            if (distances[from] !== Infinity) {
                const newDist = distances[from] + weight;
                
                // RELAXATION: Update if shorter path found
                if (newDist < distances[to]) {
                    distances[to] = newDist;
                    previous[to] = from;  // Track path
                }
            }
        }
    }
    
    // NEGATIVE CYCLE DETECTION: One more iteration
    // WHY? If distances still decrease after V-1 iterations,
    // there must be a negative cycle (infinite improvement)
    for (const edge of edges) {
        const { from, to, weight } = edge;
        if (distances[from] !== Infinity && 
            distances[from] + weight < distances[to]) {
            // Still improving = negative cycle exists
            throw new Error('Graph contains negative cycle');
        }
    }
    
    return { distances, previous };
}

// WHY BELLMAN-FORD WORKS:
// 1. Dynamic Programming: Builds up shortest paths incrementally
// 2. Considers all edges: Unlike Dijkstra's greedy approach
// 3. Convergence: After k iterations, all k-edge paths are optimal
// 4. Negative cycles: Detected when improvements continue past V-1
```

### Floyd-Warshall Algorithm
```javascript
// ============= FLOYD-WARSHALL - ALL PAIRS SHORTEST PATH =============
// Time: O(V³) - Three nested loops over vertices
// Space: O(V²) - Distance and next matrices
// USE CASE: When you need distances between ALL pairs of vertices
// Better than running Dijkstra V times for dense graphs
function floydWarshall(graph) {
    const n = graph.length;
    const dist = [];  // Distance matrix
    const next = [];  // For path reconstruction
    
    // INITIALIZATION: Set up initial distances
    for (let i = 0; i < n; i++) {
        dist[i] = [];
        next[i] = [];
        for (let j = 0; j < n; j++) {
            if (i === j) {
                // Distance to self is 0
                dist[i][j] = 0;
                next[i][j] = null;
            } else if (graph[i][j] !== undefined) {
                // Direct edge exists
                dist[i][j] = graph[i][j];
                next[i][j] = j;  // Go directly to j
            } else {
                // No direct edge
                dist[i][j] = Infinity;
                next[i][j] = null;
            }
        }
    }
    
    // DYNAMIC PROGRAMMING: Consider each vertex as intermediate
    // KEY INSIGHT: dist[i][j] through vertices 0...k
    // Can either use vertex k or not use it
    for (let k = 0; k < n; k++) {
        // For each pair of vertices
        for (let i = 0; i < n; i++) {
            for (let j = 0; j < n; j++) {
                // Check if path through k is shorter
                // dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                    // Update next vertex in path
                    // To go from i to j, first go to where i→k path goes
                    next[i][j] = next[i][k];
                }
            }
        }
    }
    
    // PATH RECONSTRUCTION helper function
    const getPath = (i, j) => {
        if (next[i][j] === null) return [];  // No path exists
        
        const path = [i];  // Start with source
        // Follow next pointers until destination
        while (i !== j) {
            i = next[i][j];  // Move to next vertex in path
            path.push(i);
        }
        return path;
    };
    
    return { dist, getPath };
}

// WHY FLOYD-WARSHALL WORKS:
// 1. Optimal Substructure: Shortest path i→j through k uses
//    shortest paths i→k and k→j
// 2. Overlapping Subproblems: Same subpaths used multiple times
// 3. Bottom-up DP: Builds from smaller to larger vertex sets
// 4. Handles negative weights (but not negative cycles)
```

## Minimum Spanning Tree

### Kruskal's Algorithm
```javascript
// ============= UNION-FIND (DISJOINT SET) DATA STRUCTURE =============
// Used for efficient cycle detection in Kruskal's
// Operations nearly O(1) with optimizations
class UnionFind {
    constructor(n) {
        // Initially, each vertex is its own parent (separate sets)
        this.parent = Array(n).fill(0).map((_, i) => i);
        // Rank for union by rank optimization
        this.rank = Array(n).fill(0);
    }
    
    // FIND with PATH COMPRESSION
    // Time: O(α(n)) ≈ O(1) amortized (α is inverse Ackermann)
    find(x) {
        // Path compression: Make all nodes point directly to root
        // WHY? Flattens tree, speeds up future operations
        if (this.parent[x] !== x) {
            // Recursively find root and compress path
            this.parent[x] = this.find(this.parent[x]);
        }
        return this.parent[x];
    }
    
    // UNION BY RANK
    // Time: O(α(n)) ≈ O(1) amortized
    union(x, y) {
        const rootX = this.find(x);
        const rootY = this.find(y);
        
        // Already in same set (would create cycle)
        if (rootX === rootY) return false;
        
        // UNION BY RANK: Attach smaller tree under larger
        // WHY? Keeps tree shallow, improves find performance
        if (this.rank[rootX] < this.rank[rootY]) {
            this.parent[rootX] = rootY;
        } else if (this.rank[rootX] > this.rank[rootY]) {
            this.parent[rootY] = rootX;
        } else {
            // Equal rank: arbitrary choice, increment rank
            this.parent[rootY] = rootX;
            this.rank[rootX]++;
        }
        
        return true;  // Successfully unioned
    }
}

// ============= KRUSKAL'S MST ALGORITHM =============
// Time: O(E log E) - dominated by sorting
// Space: O(V) for union-find
// GREEDY: Always pick minimum weight edge that doesn't create cycle
function kruskal(vertices, edges) {
    const mst = [];  // Minimum spanning tree edges
    let totalWeight = 0;
    
    // STEP 1: Sort edges by weight (greedy choice)
    // O(E log E) - dominates overall complexity
    edges.sort((a, b) => a.weight - b.weight);
    
    // STEP 2: Initialize union-find for cycle detection
    const uf = new UnionFind(vertices);
    
    // STEP 3: Process edges in order
    for (const edge of edges) {
        // Try to union the vertices
        // Returns false if they're already connected (would create cycle)
        if (uf.union(edge.from, edge.to)) {
            mst.push(edge);
            totalWeight += edge.weight;
            
            // OPTIMIZATION: MST has exactly V-1 edges
            // Stop early when we have enough edges
            if (mst.length === vertices - 1) break;
        }
        // If union returns false, skip edge (would create cycle)
    }
    
    return { mst, totalWeight };
}

// WHY KRUSKAL'S WORKS:
// 1. Cut Property: Minimum edge crossing any cut belongs to MST
// 2. Greedy Choice: Always safe to add minimum edge that doesn't cycle
// 3. Union-Find: Efficiently tracks connected components
// 4. Result: Minimum total weight spanning all vertices
```

### Prim's Algorithm
```javascript
/**
 * MST starting from a vertex
 * Time: O((V + E) log V) with min heap
 * Space: O(V)
 */
function prim(graph, start) {
    const mst = [];
    const visited = new Set();
    const minHeap = new PriorityQueue();
    let totalWeight = 0;
    
    // Start from given vertex
    visited.add(start);
    
    // Add all edges from start to heap
    for (const edge of graph.adjacencyList.get(start)) {
        minHeap.enqueue(
            { from: start, to: edge.node, weight: edge.weight },
            edge.weight
        );
    }
    
    while (!minHeap.isEmpty() && mst.length < graph.size - 1) {
        const edge = minHeap.dequeue();
        
        if (!visited.has(edge.to)) {
            visited.add(edge.to);
            mst.push(edge);
            totalWeight += edge.weight;
            
            // Add new edges to heap
            for (const nextEdge of graph.adjacencyList.get(edge.to)) {
                if (!visited.has(nextEdge.node)) {
                    minHeap.enqueue(
                        { from: edge.to, to: nextEdge.node, weight: nextEdge.weight },
                        nextEdge.weight
                    );
                }
            }
        }
    }
    
    return { mst, totalWeight };
}
```

## Advanced Graph Algorithms

### Topological Sort
```javascript
/**
 * Order vertices in DAG
 * Time: O(V + E), Space: O(V)
 */
class DirectedGraph {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(from, to) {
        this.addVertex(from);
        this.addVertex(to);
        this.adjacencyList.get(from).push(to);
    }
    
    // DFS-based topological sort
    topologicalSortDFS() {
        const visited = new Set();
        const stack = [];
        
        const dfs = (vertex) => {
            visited.add(vertex);
            
            for (const neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    dfs(neighbor);
                }
            }
            
            stack.push(vertex);
        };
        
        for (const vertex of this.adjacencyList.keys()) {
            if (!visited.has(vertex)) {
                dfs(vertex);
            }
        }
        
        return stack.reverse();
    }
    
    // Kahn's algorithm (BFS-based)
    topologicalSortKahn() {
        const inDegree = new Map();
        const queue = [];
        const result = [];
        
        // Calculate in-degrees
        for (const vertex of this.adjacencyList.keys()) {
            inDegree.set(vertex, 0);
        }
        
        for (const vertex of this.adjacencyList.keys()) {
            for (const neighbor of this.adjacencyList.get(vertex)) {
                inDegree.set(neighbor, inDegree.get(neighbor) + 1);
            }
        }
        
        // Find vertices with no incoming edges
        for (const [vertex, degree] of inDegree) {
            if (degree === 0) {
                queue.push(vertex);
            }
        }
        
        // Process vertices
        while (queue.length) {
            const vertex = queue.shift();
            result.push(vertex);
            
            for (const neighbor of this.adjacencyList.get(vertex)) {
                inDegree.set(neighbor, inDegree.get(neighbor) - 1);
                if (inDegree.get(neighbor) === 0) {
                    queue.push(neighbor);
                }
            }
        }
        
        // Check for cycle
        if (result.length !== this.adjacencyList.size) {
            throw new Error('Graph has a cycle');
        }
        
        return result;
    }
}
```

### Strongly Connected Components (Kosaraju's Algorithm)
```javascript
/**
 * Find SCCs in directed graph
 * Time: O(V + E), Space: O(V)
 */
function kosaraju(graph) {
    const visited = new Set();
    const stack = [];
    const sccs = [];
    
    // First DFS to fill stack
    const dfs1 = (vertex) => {
        visited.add(vertex);
        for (const neighbor of graph.adjacencyList.get(vertex)) {
            if (!visited.has(neighbor)) {
                dfs1(neighbor);
            }
        }
        stack.push(vertex);
    };
    
    // Perform first DFS
    for (const vertex of graph.adjacencyList.keys()) {
        if (!visited.has(vertex)) {
            dfs1(vertex);
        }
    }
    
    // Create transpose graph
    const transpose = new Map();
    for (const vertex of graph.adjacencyList.keys()) {
        transpose.set(vertex, []);
    }
    for (const [vertex, neighbors] of graph.adjacencyList) {
        for (const neighbor of neighbors) {
            transpose.get(neighbor).push(vertex);
        }
    }
    
    // Second DFS on transpose
    visited.clear();
    
    const dfs2 = (vertex, component) => {
        visited.add(vertex);
        component.push(vertex);
        for (const neighbor of transpose.get(vertex)) {
            if (!visited.has(neighbor)) {
                dfs2(neighbor, component);
            }
        }
    };
    
    // Process vertices in stack order
    while (stack.length) {
        const vertex = stack.pop();
        if (!visited.has(vertex)) {
            const component = [];
            dfs2(vertex, component);
            sccs.push(component);
        }
    }
    
    return sccs;
}
```

## AWS Applications

### 1. VPC Network Path Finding
```javascript
/**
 * Find optimal network path in VPC
 */
class VPCNetworkRouter {
    constructor() {
        this.network = new WeightedGraph();
        this.securityGroups = new Map();
    }
    
    addSubnet(id, cidr, availabilityZone) {
        this.network.addVertex(id);
        this.securityGroups.set(id, {
            cidr,
            availabilityZone,
            rules: []
        });
    }
    
    addRoute(from, to, type = 'direct') {
        // Weight based on connection type
        const weights = {
            'direct': 1,
            'peering': 2,
            'transit-gateway': 3,
            'vpn': 5,
            'internet': 10
        };
        
        const weight = weights[type] || 100;
        this.network.addEdge(from, to, weight);
    }
    
    findOptimalPath(source, destination) {
        return this.network.dijkstra(source, destination);
    }
    
    // Check if path respects security groups
    validatePath(path) {
        for (let i = 0; i < path.length - 1; i++) {
            const from = path[i];
            const to = path[i + 1];
            
            if (!this.isConnectionAllowed(from, to)) {
                return {
                    valid: false,
                    blockedAt: { from, to }
                };
            }
        }
        
        return { valid: true };
    }
    
    isConnectionAllowed(from, to) {
        const fromSG = this.securityGroups.get(from);
        const toSG = this.securityGroups.get(to);
        
        // Check security group rules (simplified)
        for (const rule of toSG.rules) {
            if (rule.type === 'ingress' && 
                this.cidrContains(rule.source, fromSG.cidr)) {
                return true;
            }
        }
        
        return false;
    }
    
    cidrContains(cidr1, cidr2) {
        // Simplified CIDR check
        return true; // Actual implementation would check IP ranges
    }
}
```

### 2. Service Dependency Resolution
```javascript
/**
 * Resolve service dependencies for deployment
 */
class ServiceDependencyResolver {
    constructor() {
        this.services = new DirectedGraph();
        this.metadata = new Map();
    }
    
    addService(name, config) {
        this.services.addVertex(name);
        this.metadata.set(name, config);
    }
    
    addDependency(service, dependsOn) {
        this.services.addEdge(dependsOn, service);
    }
    
    getDeploymentOrder() {
        try {
            return this.services.topologicalSortKahn();
        } catch (error) {
            // Detect circular dependencies
            const cycles = this.findCycles();
            throw new Error(`Circular dependencies detected: ${JSON.stringify(cycles)}`);
        }
    }
    
    findCycles() {
        const visited = new Set();
        const recStack = new Set();
        const cycles = [];
        
        const dfs = (vertex, path = []) => {
            visited.add(vertex);
            recStack.add(vertex);
            path.push(vertex);
            
            for (const neighbor of this.services.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    dfs(neighbor, [...path]);
                } else if (recStack.has(neighbor)) {
                    // Found cycle
                    const cycleStart = path.indexOf(neighbor);
                    cycles.push(path.slice(cycleStart));
                }
            }
            
            recStack.delete(vertex);
        };
        
        for (const vertex of this.services.adjacencyList.keys()) {
            if (!visited.has(vertex)) {
                dfs(vertex);
            }
        }
        
        return cycles;
    }
    
    // Parallel deployment for independent services
    getParallelDeploymentPlan() {
        const order = this.getDeploymentOrder();
        const levels = [];
        const deployed = new Set();
        
        while (deployed.size < order.length) {
            const level = [];
            
            for (const service of order) {
                if (deployed.has(service)) continue;
                
                // Check if all dependencies are deployed
                const ready = this.getDependencies(service)
                    .every(dep => deployed.has(dep));
                
                if (ready) {
                    level.push(service);
                }
            }
            
            for (const service of level) {
                deployed.add(service);
            }
            
            levels.push(level);
        }
        
        return levels;
    }
    
    getDependencies(service) {
        const deps = [];
        
        for (const [vertex, neighbors] of this.services.adjacencyList) {
            if (neighbors.includes(service)) {
                deps.push(vertex);
            }
        }
        
        return deps;
    }
}
```

### 3. Cost Optimization Network
```javascript
/**
 * Optimize data transfer costs in AWS
 */
class DataTransferOptimizer {
    constructor() {
        this.regions = new WeightedGraph();
        this.costs = new Map();
    }
    
    addRegion(name, location) {
        this.regions.addVertex(name);
        this.costs.set(name, { location, services: [] });
    }
    
    addTransferCost(from, to, costPerGB) {
        this.regions.addEdge(from, to, costPerGB);
    }
    
    // Find cheapest path for data transfer
    findCheapestRoute(source, destination, dataSize) {
        const result = this.regions.dijkstra(source, destination);
        
        return {
            path: result.path,
            totalCost: result.distance * dataSize,
            costPerGB: result.distance
        };
    }
    
    // Minimum cost network for multi-region replication
    buildReplicationNetwork(regions) {
        const edges = [];
        
        // Create complete graph
        for (let i = 0; i < regions.length; i++) {
            for (let j = i + 1; j < regions.length; j++) {
                const cost = this.getTransferCost(regions[i], regions[j]);
                edges.push({
                    from: i,
                    to: j,
                    weight: cost
                });
            }
        }
        
        // Find MST for minimum cost network
        const mst = kruskal(regions.length, edges);
        
        return {
            connections: mst.mst.map(edge => ({
                from: regions[edge.from],
                to: regions[edge.to],
                costPerGB: edge.weight
            })),
            totalMonthlyCost: this.estimateMonthlyCost(mst.totalWeight)
        };
    }
    
    getTransferCost(region1, region2) {
        // Simplified cost calculation
        const sameContinent = this.costs.get(region1).location === 
                            this.costs.get(region2).location;
        
        return sameContinent ? 0.02 : 0.09; // Per GB
    }
    
    estimateMonthlyCost(costPerGB, avgMonthlyData = 1000) {
        return costPerGB * avgMonthlyData;
    }
}
```

## Practice Problems

### Problem 1: Network Delay Time
```javascript
/**
 * Find time for signal to reach all nodes
 * Time: O((V + E) log V), Space: O(V)
 */
function networkDelayTime(times, n, k) {
    const graph = new Map();
    
    // Build graph
    for (let i = 1; i <= n; i++) {
        graph.set(i, []);
    }
    
    for (const [u, v, w] of times) {
        graph.get(u).push({ node: v, time: w });
    }
    
    // Dijkstra from source k
    const distances = new Map();
    const pq = [[k, 0]]; // [node, time]
    
    while (pq.length) {
        pq.sort((a, b) => a[1] - b[1]);
        const [node, time] = pq.shift();
        
        if (distances.has(node)) continue;
        distances.set(node, time);
        
        for (const { node: neighbor, time: edgeTime } of graph.get(node)) {
            if (!distances.has(neighbor)) {
                pq.push([neighbor, time + edgeTime]);
            }
        }
    }
    
    // Check if all nodes reached
    if (distances.size !== n) return -1;
    
    return Math.max(...distances.values());
}
```

### Problem 2: Critical Connections (Bridges)
```javascript
/**
 * Find bridges in network
 * Time: O(V + E), Space: O(V)
 */
function criticalConnections(n, connections) {
    const graph = Array(n).fill(null).map(() => []);
    
    for (const [u, v] of connections) {
        graph[u].push(v);
        graph[v].push(u);
    }
    
    const bridges = [];
    const disc = Array(n).fill(-1);
    const low = Array(n).fill(-1);
    let time = 0;
    
    const dfs = (u, parent) => {
        disc[u] = low[u] = time++;
        
        for (const v of graph[u]) {
            if (v === parent) continue;
            
            if (disc[v] === -1) {
                dfs(v, u);
                low[u] = Math.min(low[u], low[v]);
                
                // Bridge found
                if (low[v] > disc[u]) {
                    bridges.push([u, v]);
                }
            } else {
                low[u] = Math.min(low[u], disc[v]);
            }
        }
    };
    
    for (let i = 0; i < n; i++) {
        if (disc[i] === -1) {
            dfs(i, -1);
        }
    }
    
    return bridges;
}
```

## Interview Tips

### Common Questions
1. **"When to use BFS vs DFS?"**
   - BFS: Shortest path in unweighted graph, level-order
   - DFS: Cycle detection, topological sort, backtracking
   - BFS uses queue, DFS uses stack/recursion

2. **"Explain Dijkstra vs Bellman-Ford"**
   - Dijkstra: Faster O((V+E)log V), no negative weights
   - Bellman-Ford: Slower O(VE), handles negative weights
   - Bellman-Ford can detect negative cycles

3. **"How to handle large graphs?"**
   - Use adjacency list for sparse graphs
   - Consider distributed algorithms
   - Implement incremental processing

### Red Flags to Avoid
- ❌ Using Dijkstra with negative weights
- ❌ Not checking for disconnected components
- ❌ Forgetting to mark vertices as visited
- ❌ Not handling cycles in directed graphs

### AWS-Specific Follow-ups
- "Design VPC routing optimization"
- "Implement service mesh path finding"
- "Optimize multi-region data replication"
- "Build dependency graph for microservices"

## Key Takeaways
1. **BFS** for shortest path in unweighted graphs
2. **DFS** for exploring all paths and detecting cycles
3. **Dijkstra** for weighted shortest path without negative edges
4. **Union-Find** for connectivity and MST problems
5. **Topological sort** for dependency resolution in DAGs

---

*Previous: [← Divide and Conquer](./06-divide-and-conquer.md)*
*Next: [Scalability Patterns →](./08-scalability-patterns.md)*
