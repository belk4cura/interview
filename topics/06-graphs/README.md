# 📊 Graphs

## Overview
Graphs are non-linear data structures consisting of vertices (nodes) connected by edges. They model relationships and networks, from social connections to road maps, making them essential for many real-world problems.

## Key Concepts

### Graph Types
- **Directed vs Undirected**: One-way vs two-way edges
- **Weighted vs Unweighted**: Edges with or without costs
- **Cyclic vs Acyclic**: Contains cycles or not (DAG)
- **Connected vs Disconnected**: All vertices reachable or not
- **Dense vs Sparse**: Many edges vs few edges

### Graph Representations
| Method | Space | Lookup | Iterate Neighbors | Best For |
|--------|-------|--------|------------------|----------|
| Adjacency Matrix | O(V²) | O(1) | O(V) | Dense graphs |
| Adjacency List | O(V+E) | O(V) | O(degree) | Sparse graphs |
| Edge List | O(E) | O(E) | O(E) | Kruskal's algorithm |

## Common Algorithms

### Traversal
```javascript
// DFS - Depth First Search
function dfs(graph, start, visited = new Set()) {
    visited.add(start);
    console.log(start);
    
    for (const neighbor of graph[start]) {
        if (!visited.has(neighbor)) {
            dfs(graph, neighbor, visited);
        }
    }
}

// BFS - Breadth First Search
function bfs(graph, start) {
    const visited = new Set([start]);
    const queue = [start];
    
    while (queue.length) {
        const vertex = queue.shift();
        console.log(vertex);
        
        for (const neighbor of graph[vertex]) {
            if (!visited.has(neighbor)) {
                visited.add(neighbor);
                queue.push(neighbor);
            }
        }
    }
}
```

### Shortest Path
```javascript
// Dijkstra's Algorithm (weighted, non-negative)
function dijkstra(graph, start) {
    const distances = {};
    const visited = new Set();
    const pq = [[0, start]]; // [distance, node]
    
    // Initialize distances
    for (const node in graph) {
        distances[node] = Infinity;
    }
    distances[start] = 0;
    
    while (pq.length) {
        pq.sort((a, b) => a[0] - b[0]);
        const [dist, node] = pq.shift();
        
        if (visited.has(node)) continue;
        visited.add(node);
        
        for (const [neighbor, weight] of graph[node]) {
            const newDist = dist + weight;
            if (newDist < distances[neighbor]) {
                distances[neighbor] = newDist;
                pq.push([newDist, neighbor]);
            }
        }
    }
    
    return distances;
}
```

## Problems in This Section

### Basic
- [BFS Shortest Path](problems/bfs-shortest-path.md) - Unweighted shortest path
- [Snakes and Ladders](problems/snakes-and-ladders.md) - Board game BFS

### Intermediate
- [Clone Graph](problems/clone-graph.md) - Deep copy with cycles
- [Course Schedule](problems/course-schedule.md) - Topological sort
- [Network Delay Time](problems/network-delay-time.md) - Dijkstra's algorithm

### Advanced
- [Critical Connections](problems/critical-connections.md) - Find bridges

## Common Patterns

### 1. Graph Building
```javascript
// From edges list
function buildGraph(edges, directed = false) {
    const graph = {};
    
    for (const [u, v] of edges) {
        if (!graph[u]) graph[u] = [];
        if (!graph[v]) graph[v] = [];
        
        graph[u].push(v);
        if (!directed) {
            graph[v].push(u);
        }
    }
    
    return graph;
}
```

### 2. Cycle Detection
```javascript
// Directed Graph (DFS with colors)
function hasCycle(graph) {
    const color = {}; // 0: white, 1: gray, 2: black
    
    function dfs(node) {
        color[node] = 1; // Gray (visiting)
        
        for (const neighbor of (graph[node] || [])) {
            if (color[neighbor] === 1) return true; // Back edge
            if (!color[neighbor] && dfs(neighbor)) return true;
        }
        
        color[node] = 2; // Black (visited)
        return false;
    }
    
    for (const node in graph) {
        if (!color[node] && dfs(node)) return true;
    }
    
    return false;
}

// Undirected Graph (Union-Find)
class UnionFind {
    constructor(n) {
        this.parent = Array.from({length: n}, (_, i) => i);
        this.rank = new Array(n).fill(0);
    }
    
    find(x) {
        if (this.parent[x] !== x) {
            this.parent[x] = this.find(this.parent[x]);
        }
        return this.parent[x];
    }
    
    union(x, y) {
        const rootX = this.find(x);
        const rootY = this.find(y);
        
        if (rootX === rootY) return false;
        
        if (this.rank[rootX] < this.rank[rootY]) {
            this.parent[rootX] = rootY;
        } else if (this.rank[rootX] > this.rank[rootY]) {
            this.parent[rootY] = rootX;
        } else {
            this.parent[rootY] = rootX;
            this.rank[rootX]++;
        }
        
        return true;
    }
}
```

### 3. Topological Sort
```javascript
// Kahn's Algorithm (BFS)
function topologicalSort(graph, numNodes) {
    const indegree = new Array(numNodes).fill(0);
    const result = [];
    
    // Calculate indegrees
    for (const node in graph) {
        for (const neighbor of graph[node]) {
            indegree[neighbor]++;
        }
    }
    
    // Find nodes with no dependencies
    const queue = [];
    for (let i = 0; i < numNodes; i++) {
        if (indegree[i] === 0) queue.push(i);
    }
    
    // Process nodes
    while (queue.length) {
        const node = queue.shift();
        result.push(node);
        
        for (const neighbor of (graph[node] || [])) {
            indegree[neighbor]--;
            if (indegree[neighbor] === 0) {
                queue.push(neighbor);
            }
        }
    }
    
    return result.length === numNodes ? result : [];
}
```

### 4. Connected Components
```javascript
// Find all connected components
function findComponents(graph) {
    const visited = new Set();
    const components = [];
    
    function dfs(node, component) {
        visited.add(node);
        component.push(node);
        
        for (const neighbor of (graph[node] || [])) {
            if (!visited.has(neighbor)) {
                dfs(neighbor, component);
            }
        }
    }
    
    for (const node in graph) {
        if (!visited.has(node)) {
            const component = [];
            dfs(node, component);
            components.push(component);
        }
    }
    
    return components;
}
```

## Advanced Algorithms

### Minimum Spanning Tree
```javascript
// Kruskal's Algorithm
function kruskal(edges, n) {
    edges.sort((a, b) => a[2] - b[2]); // Sort by weight
    const uf = new UnionFind(n);
    const mst = [];
    let cost = 0;
    
    for (const [u, v, weight] of edges) {
        if (uf.union(u, v)) {
            mst.push([u, v, weight]);
            cost += weight;
        }
    }
    
    return {mst, cost};
}
```

## Tips for Success

1. **Choose right representation** - Matrix vs List based on density
2. **Handle disconnected graphs** - Multiple components
3. **Watch for cycles** - Different for directed/undirected
4. **Consider edge cases** - Self-loops, parallel edges
5. **Track visited nodes** - Avoid infinite loops
6. **Understand problem type** - Path, connectivity, optimization?

## Common Interview Questions

- Find shortest path (BFS/Dijkstra)
- Detect cycles in directed/undirected graph
- Find connected components
- Topological ordering
- Minimum spanning tree
- Is graph bipartite?
- Find bridges and articulation points

## When to Use Graphs

✅ **Perfect for:**
- Network problems (social, computer, road)
- Dependency resolution
- Path finding
- State machines
- Relationship modeling

❌ **Avoid when:**
- Simple linear relationships (use array/list)
- Hierarchical only (use tree)
- No relationships between elements
