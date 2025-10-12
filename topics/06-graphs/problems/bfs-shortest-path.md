# BFS Shortest Path in Graph

## 🎯 What Are We Trying To Solve?

Finding the shortest path in an unweighted graph is like finding the quickest route through a city where every street takes the same time to walk. BFS guarantees we find the shortest path because it explores all nodes at distance k before exploring nodes at distance k+1.

## 📝 The Problem

Given an undirected graph and a starting node, find the shortest distance from the start node to all other nodes. If a node is unreachable, return -1. In this problem, each edge has weight 6 (or consider it as unit weight for BFS).

```
Example:
Graph: 1 -- 2 -- 3
       |         |
       4 -------- 5
       
Start: 1
Distances: 
- To node 2: 1 edge × 6 = 6
- To node 3: 2 edges × 6 = 12
- To node 4: 1 edge × 6 = 6
- To node 5: 2 edges × 6 = 12
```

## 💻 The Code

```javascript
/**
 * BFS Shortest Path for unweighted graph
 * @param {number} n - Number of nodes (1 to n)
 * @param {number[][]} edges - Array of [u, v] edges
 * @param {number} start - Starting node
 * @returns {number[]} - Shortest distances (×6) to all nodes except start
 */
function bfsShortestReach(n, edges, start) {
    // Build adjacency list
    const graph = new Map();
    for (let i = 1; i <= n; i++) {
        graph.set(i, new Set());
    }
    
    // Add edges (undirected)
    for (const [u, v] of edges) {
        graph.get(u).add(v);
        graph.get(v).add(u);
    }
    
    // BFS from start node
    const distances = new Array(n + 1).fill(-1);
    distances[start] = 0;
    
    const queue = [start];
    
    while (queue.length > 0) {
        const current = queue.shift();
        
        // Visit all neighbors
        for (const neighbor of graph.get(current)) {
            if (distances[neighbor] === -1) {
                distances[neighbor] = distances[current] + 1;
                queue.push(neighbor);
            }
        }
    }
    
    // Format output (exclude start node, multiply by 6)
    const result = [];
    for (let i = 1; i <= n; i++) {
        if (i !== start) {
            result.push(distances[i] === -1 ? -1 : distances[i] * 6);
        }
    }
    
    return result;
}

/**
 * Alternative: Using visited set instead of distance array
 */
function bfsWithVisited(n, edges, start) {
    // Build graph
    const graph = Array.from({ length: n + 1 }, () => []);
    for (const [u, v] of edges) {
        graph[u].push(v);
        graph[v].push(u);
    }
    
    const visited = new Set([start]);
    const distances = new Map([[start, 0]]);
    const queue = [start];
    
    while (queue.length > 0) {
        const current = queue.shift();
        const currentDist = distances.get(current);
        
        for (const neighbor of graph[current]) {
            if (!visited.has(neighbor)) {
                visited.add(neighbor);
                distances.set(neighbor, currentDist + 1);
                queue.push(neighbor);
            }
        }
    }
    
    // Build result
    const result = [];
    for (let i = 1; i <= n; i++) {
        if (i !== start) {
            const dist = distances.get(i);
            result.push(dist === undefined ? -1 : dist * 6);
        }
    }
    
    return result;
}

/**
 * Level-order BFS (process level by level)
 */
function bfsLevelOrder(n, edges, start) {
    // Build adjacency list
    const graph = new Map();
    for (let i = 1; i <= n; i++) {
        graph.set(i, []);
    }
    
    for (const [u, v] of edges) {
        graph.get(u).push(v);
        graph.get(v).push(u);
    }
    
    const distances = new Array(n + 1).fill(-1);
    distances[start] = 0;
    
    let queue = [start];
    let level = 1;
    
    while (queue.length > 0) {
        const nextQueue = [];
        
        // Process all nodes at current level
        for (const node of queue) {
            for (const neighbor of graph.get(node)) {
                if (distances[neighbor] === -1) {
                    distances[neighbor] = level;
                    nextQueue.push(neighbor);
                }
            }
        }
        
        queue = nextQueue;
        level++;
    }
    
    // Format result
    const result = [];
    for (let i = 1; i <= n; i++) {
        if (i !== start) {
            result.push(distances[i] === -1 ? -1 : distances[i] * 6);
        }
    }
    
    return result;
}

/**
 * Multi-source BFS (find shortest path from multiple sources)
 */
function multiSourceBFS(n, edges, sources) {
    // Build graph
    const graph = Array.from({ length: n + 1 }, () => []);
    for (const [u, v] of edges) {
        graph[u].push(v);
        graph[v].push(u);
    }
    
    const distances = new Array(n + 1).fill(-1);
    const queue = [];
    
    // Initialize all sources
    for (const source of sources) {
        distances[source] = 0;
        queue.push(source);
    }
    
    // BFS from all sources simultaneously
    while (queue.length > 0) {
        const current = queue.shift();
        
        for (const neighbor of graph[current]) {
            if (distances[neighbor] === -1) {
                distances[neighbor] = distances[current] + 1;
                queue.push(neighbor);
            }
        }
    }
    
    return distances.slice(1); // Exclude index 0
}

/**
 * Find shortest path and reconstruct it
 */
function bfsWithPath(n, edges, start, end) {
    // Build graph
    const graph = new Map();
    for (let i = 1; i <= n; i++) {
        graph.set(i, []);
    }
    
    for (const [u, v] of edges) {
        graph.get(u).push(v);
        graph.get(v).push(u);
    }
    
    const distances = new Map([[start, 0]]);
    const parent = new Map([[start, null]]);
    const queue = [start];
    
    while (queue.length > 0) {
        const current = queue.shift();
        
        if (current === end) break;
        
        for (const neighbor of graph.get(current)) {
            if (!distances.has(neighbor)) {
                distances.set(neighbor, distances.get(current) + 1);
                parent.set(neighbor, current);
                queue.push(neighbor);
            }
        }
    }
    
    // Reconstruct path
    if (!parent.has(end)) {
        return { distance: -1, path: [] };
    }
    
    const path = [];
    let current = end;
    while (current !== null) {
        path.unshift(current);
        current = parent.get(current);
    }
    
    return {
        distance: distances.get(end) * 6,
        path: path
    };
}

// Test cases
const n1 = 4;
const edges1 = [[1, 2], [1, 3]];
const start1 = 1;
console.log(bfsShortestReach(n1, edges1, start1)); 
// [6, 6, -1] (distances to nodes 2, 3, 4)

const n2 = 5;
const edges2 = [[1, 2], [2, 3], [3, 4], [1, 4], [4, 5]];
const start2 = 1;
console.log(bfsShortestReach(n2, edges2, start2));
// [6, 12, 6, 12] (distances to nodes 2, 3, 4, 5)
```

## 🎨 Step-by-Step Walkthrough

Let's trace BFS from node 1:

```
Graph:  1 -- 2 -- 3
        |         |
        4 ------- 5

Step 1: Start at node 1
Queue: [1]
Distances: {1: 0}

Step 2: Process node 1
Neighbors: [2, 4]
Queue: [2, 4]
Distances: {1: 0, 2: 1, 4: 1}

Step 3: Process node 2
Neighbors: [1, 3] (1 already visited)
Queue: [4, 3]
Distances: {1: 0, 2: 1, 3: 2, 4: 1}

Step 4: Process node 4
Neighbors: [1, 5] (1 already visited)
Queue: [3, 5]
Distances: {1: 0, 2: 1, 3: 2, 4: 1, 5: 2}

Step 5: Process node 3
Neighbors: [2, 5] (both already visited)
Queue: [5]

Step 6: Process node 5
Neighbors: [3, 4] (both already visited)
Queue: []

Final distances × 6: [6, 12, 6, 12]
```

## 📊 Visual Understanding

```
BFS Wave Propagation:

Level 0: [1]        Start node
         ↓
Level 1: [2,4]      Distance = 1
         ↓
Level 2: [3,5]      Distance = 2

Graph Exploration:
    1 (0)
   /│\
  2 │ 4 (1)
  │ │ │
  3 └─5 (2)

Distance Calculation:
Node 1 → Node 2: 1 hop × 6 = 6
Node 1 → Node 3: 2 hops × 6 = 12
Node 1 → Node 4: 1 hop × 6 = 6
Node 1 → Node 5: 2 hops × 6 = 12
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(V + E)
  - Visit each vertex once: O(V)
  - Explore each edge once: O(E)
  
- **Space Complexity:** O(V)
  - Queue can contain at most V vertices
  - Distance/visited arrays: O(V)

## 🚫 Common Mistakes

1. **Not handling disconnected components**
   ```javascript
   // Wrong: Assumes all nodes are reachable
   return distances[i] * 6;
   
   // Right: Check if node was visited
   return distances[i] === -1 ? -1 : distances[i] * 6;
   ```

2. **Processing nodes multiple times**
   ```javascript
   // Wrong: No visited check
   for (const neighbor of graph[current]) {
       queue.push(neighbor);
   }
   
   // Right: Check if already visited
   if (distances[neighbor] === -1) {
       distances[neighbor] = distances[current] + 1;
       queue.push(neighbor);
   }
   ```

3. **Incorrect edge weight application**
   ```javascript
   // Wrong: Multiply during BFS
   distances[neighbor] = distances[current] + 6;
   
   // Right: Track hops, multiply at the end
   distances[neighbor] = distances[current] + 1;
   // Later: return distances[i] * 6;
   ```

## 🔄 Variations

```javascript
// Bidirectional BFS (faster for finding path between two nodes)
function bidirectionalBFS(n, edges, start, end) {
    const graph = buildGraph(n, edges);
    
    const visitedStart = new Map([[start, 0]]);
    const visitedEnd = new Map([[end, 0]]);
    let queueStart = [start];
    let queueEnd = [end];
    
    while (queueStart.length > 0 && queueEnd.length > 0) {
        // Expand from smaller frontier
        if (queueStart.length <= queueEnd.length) {
            const result = expandLevel(graph, queueStart, visitedStart, visitedEnd);
            if (result !== null) return result;
        } else {
            const result = expandLevel(graph, queueEnd, visitedEnd, visitedStart);
            if (result !== null) return result;
        }
    }
    
    return -1;
}

// BFS with edge weights (requires priority queue)
function bfsWeighted(n, weightedEdges, start) {
    // This becomes Dijkstra's algorithm
    const pq = new MinPriorityQueue();
    const distances = new Array(n + 1).fill(Infinity);
    
    pq.enqueue(start, 0);
    distances[start] = 0;
    
    while (!pq.isEmpty()) {
        const { element: node, priority: dist } = pq.dequeue();
        
        if (dist > distances[node]) continue;
        
        for (const [neighbor, weight] of graph.get(node)) {
            const newDist = dist + weight;
            if (newDist < distances[neighbor]) {
                distances[neighbor] = newDist;
                pq.enqueue(neighbor, newDist);
            }
        }
    }
    
    return distances;
}
```

## 🔗 Related Problems

- **Word Ladder** - BFS on word transformations
- **01 Matrix** - Multi-source BFS
- **Shortest Path in Binary Matrix** - BFS on grid
- **Network Delay Time** - Weighted shortest path (Dijkstra)

## 💡 Key Takeaways

1. **BFS Guarantees**: Shortest path in unweighted graphs
2. **Level Processing**: Can process nodes level by level
3. **Visited Tracking**: Essential to avoid cycles
4. **Distance Calculation**: Track hops, not actual weights
5. **Disconnected Graphs**: Some nodes may be unreachable

BFS is the go-to algorithm for shortest paths in unweighted graphs!
