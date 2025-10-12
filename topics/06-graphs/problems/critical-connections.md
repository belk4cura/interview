# Critical Connections (Find Bridges)

## 🎯 What Are We Trying To Solve?

Imagine you're managing a network of servers where some connections are critical - if they fail, parts of the network become disconnected. These are called "bridges" or "critical connections". It's like finding the single roads that, if blocked, would cut off entire neighborhoods from each other!

## 📝 The Problem

There are `n` servers numbered from 0 to n-1 connected by undirected server-to-server connections forming a network. A critical connection is a connection that, if removed, will make some servers unable to reach some other server.

Return all critical connections in the network in any order.

```
Example:
Input: n = 4, connections = [[0,1],[1,2],[2,0],[1,3]]
Output: [[1,3]]

Network:
0 --- 1 --- 3
 \   /
  \ /
   2

If we remove edge [1,3], node 3 becomes isolated!
```

## 🧠 How to Think About This

Use Tarjan's algorithm with DFS! We track discovery time and "low-link" values. A connection is critical if there's no alternative path (no back edge) that can reach an ancestor without using this connection.

### Key Concepts
- **Discovery Time**: When we first visit a node
- **Low-Link Value**: Earliest visited vertex reachable from the subtree
- **Bridge Condition**: `low[v] > disc[u]` (can't reach u or earlier without this edge)

## 💻 The Code

```javascript
/**
 * Find all bridges (critical connections) in network
 * Time: O(V + E) - single DFS traversal
 * Space: O(V) - for tracking arrays
 */
function criticalConnections(n, connections) {
    // Build adjacency list graph
    const graph = Array(n).fill(null).map(() => []);
    
    for (const [u, v] of connections) {
        graph[u].push(v);
        graph[v].push(u);
    }
    
    // Tarjan's algorithm setup
    const bridges = [];
    const disc = Array(n).fill(-1);  // Discovery times
    const low = Array(n).fill(-1);   // Low-link values
    let time = 0;                    // Global timer
    
    /**
     * DFS to find bridges
     * @param u - current vertex
     * @param parent - parent in DFS tree (to avoid going back)
     */
    function dfs(u, parent) {
        // Set discovery time and initial low-link value
        disc[u] = low[u] = time++;
        
        // Explore all neighbors
        for (const v of graph[u]) {
            // Skip the edge we came from (undirected graph)
            if (v === parent) continue;
            
            if (disc[v] === -1) {
                // Unvisited neighbor - tree edge
                dfs(v, u);
                
                // Update low-link value after exploring subtree
                // Can we reach an earlier vertex through v's subtree?
                low[u] = Math.min(low[u], low[v]);
                
                // BRIDGE CONDITION:
                // If v's subtree cannot reach u or earlier
                // without using edge u-v, then u-v is a bridge
                if (low[v] > disc[u]) {
                    bridges.push([u, v]);
                }
            } else {
                // Visited neighbor - back edge
                // Update low-link to earliest reachable
                low[u] = Math.min(low[u], disc[v]);
            }
        }
    }
    
    // Start DFS from all unvisited nodes (handle disconnected graph)
    for (let i = 0; i < n; i++) {
        if (disc[i] === -1) {
            dfs(i, -1);
        }
    }
    
    return bridges;
}

// Test cases
console.log(criticalConnections(4, [[0,1],[1,2],[2,0],[1,3]])); 
// [[1,3]]

console.log(criticalConnections(2, [[0,1]])); 
// [[0,1]] - only connection is critical

console.log(criticalConnections(5, [[0,1],[1,2],[2,0],[1,3],[3,4]]));
// [[3,4],[1,3]] - two bridges
```

## 🎨 Step-by-Step Walkthrough

Let's trace through the example: `n = 4, connections = [[0,1],[1,2],[2,0],[1,3]]`

```
Graph:
0 --- 1 --- 3
 \   /
  \ /
   2

DFS from node 0:

Step 1: Visit 0
- disc[0] = 0, low[0] = 0
- Neighbors: [1, 2]

Step 2: Visit 1 from 0
- disc[1] = 1, low[1] = 1
- Neighbors: [0, 2, 3]
- Skip 0 (parent)

Step 3: Visit 2 from 1
- disc[2] = 2, low[2] = 2
- Neighbors: [1, 0]
- Skip 1 (parent)
- 0 is visited: low[2] = min(2, disc[0]) = 0

Step 4: Back to 1, update low[1]
- low[1] = min(1, low[2]) = min(1, 0) = 0
- Check bridge: low[2]=0 > disc[1]=1? NO

Step 5: Visit 3 from 1
- disc[3] = 3, low[3] = 3
- Neighbors: [1]
- Only 1 (parent), no other edges

Step 6: Back to 1, update and check bridge
- low[1] = min(0, low[3]) = min(0, 3) = 0
- Check bridge: low[3]=3 > disc[1]=1? YES!
- Bridge found: [1, 3]

Final discovery and low values:
disc = [0, 1, 2, 3]
low  = [0, 0, 0, 3]

Node 3 cannot reach anything earlier than itself
without using edge [1,3] → It's a bridge!
```

## 📊 Why This Works

### The Bridge Condition Explained

```
For edge u → v to be a bridge:
low[v] > disc[u]

This means:
- v's subtree cannot reach u or any ancestor of u
- The only way to reach v is through edge u-v
- Removing u-v disconnects v's subtree

Visual:
    u (disc=5)
    |
    v (disc=6, low=6)  ← Can't reach 5 or earlier
    |
  subtree
```

### Low-Link Value Updates

```
Two cases update low[u]:

1. Tree edge (u → v, v unvisited):
   low[u] = min(low[u], low[v])
   Inherit the earliest reachable from child

2. Back edge (u → v, v visited):
   low[u] = min(low[u], disc[v])
   Can reach v directly via back edge
```

## 🚫 Common Mistakes

1. **Not Skipping Parent Edge**
   ```javascript
   // Wrong: Treats parent edge as back edge
   for (const v of graph[u]) {
       if (disc[v] === -1) {
           dfs(v, u);
       }
   }
   
   // Right: Skip parent to avoid going back
   for (const v of graph[u]) {
       if (v === parent) continue;
       // ... rest of logic
   }
   ```

2. **Wrong Low-Link Update for Back Edges**
   ```javascript
   // Wrong: Using low[v] for back edge
   low[u] = Math.min(low[u], low[v]);
   
   // Right: Use disc[v] for back edge
   if (disc[v] === -1) {
       // Tree edge - use low[v]
       low[u] = Math.min(low[u], low[v]);
   } else {
       // Back edge - use disc[v]
       low[u] = Math.min(low[u], disc[v]);
   }
   ```

3. **Missing Bridge Condition Check**
   ```javascript
   // Wrong: Checking equality
   if (low[v] === disc[u])
   
   // Right: Strictly greater than
   if (low[v] > disc[u])
   ```

## 🔄 Alternative: Using Union-Find

```javascript
function criticalConnectionsUnionFind(n, connections) {
    const bridges = [];
    const ranks = new Map();
    
    // Try removing each edge and check connectivity
    for (let skip = 0; skip < connections.length; skip++) {
        const uf = new UnionFind(n);
        
        // Build graph without edge at index 'skip'
        for (let i = 0; i < connections.length; i++) {
            if (i !== skip) {
                const [u, v] = connections[i];
                uf.union(u, v);
            }
        }
        
        // Check if graph is still connected
        if (uf.componentCount > 1) {
            bridges.push(connections[skip]);
        }
    }
    
    return bridges;
}
// Time: O(E² α(V)) - much slower but simpler concept
```

## 🔗 Related Problems

- **Find Articulation Points** - Critical nodes instead of edges
- **Strongly Connected Components** - Similar DFS technique
- **Redundant Connection** - Find edge creating cycle
- **Minimum Spanning Tree** - Related connectivity concept

## 💡 Key Takeaways

1. **Tarjan's Algorithm**: Elegant O(V+E) solution using DFS
2. **Discovery vs Low-Link**: Track when visited vs earliest reachable
3. **Bridge Condition**: `low[v] > disc[u]` means no alternate path
4. **Handle Parent Edge**: Don't treat it as a back edge

This problem teaches a fundamental graph algorithm used in network reliability!
