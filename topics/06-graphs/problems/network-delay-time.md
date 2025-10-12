# Network Delay Time

## 🎯 What Are We Trying To Solve?

Imagine you're a network administrator sending a signal from one server that needs to reach all other servers in your network. Each connection has a delay time. You need to find out how long it takes for the signal to reach ALL servers. It's like finding the longest path in a relay race where everyone must finish!

## 📝 The Problem

You are given a network of `n` nodes labeled from 1 to n. You are also given `times`, a list of travel times as directed edges `times[i] = [ui, vi, wi]`, where ui is the source node, vi is the target node, and wi is the time it takes for a signal to travel from source to target.

Send a signal from node `k`. Return the minimum time it takes for all nodes to receive the signal. If it's impossible for all nodes to receive the signal, return -1.

```
Example:
Input: times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
Output: 2

Network:
    1
   ↗ 1
  2 → 3 → 4
    1   1

Signal reaches: node 1 at time 1, node 3 at time 1, node 4 at time 2
Maximum time = 2
```

## 🧠 How to Think About This

This is a single-source shortest path problem! Use Dijkstra's algorithm to find the shortest time to reach each node from the starting node. The answer is the maximum of all these times (the last node to receive the signal).

### Visual Process
```
Starting from node k:
1. Send signal with time 0
2. Process neighbors, updating arrival times
3. Always process the node with earliest arrival time next
4. The maximum arrival time is our answer
```

## 💻 The Code

```javascript
/**
 * Find time for signal to reach all nodes
 * Time: O((V + E) log V) with simple priority queue
 * Space: O(V) for distance tracking
 */
function networkDelayTime(times, n, k) {
    // Build adjacency list representation
    const graph = new Map();
    
    // Initialize all nodes (1 to n)
    for (let i = 1; i <= n; i++) {
        graph.set(i, []);
    }
    
    // Add edges with weights (travel times)
    for (const [source, target, time] of times) {
        graph.get(source).push({ node: target, time });
    }
    
    // Track shortest time to reach each node
    const distances = new Map();
    
    // Priority queue: [node, time to reach]
    // Start with source node k at time 0
    const pq = [[k, 0]];
    
    // Dijkstra's algorithm
    while (pq.length > 0) {
        // Sort to get minimum time (simple priority queue)
        // In production, use a proper min-heap for O(log n) operations
        pq.sort((a, b) => a[1] - b[1]);
        const [node, time] = pq.shift();
        
        // Skip if we've already processed this node
        // (We may have multiple entries for same node)
        if (distances.has(node)) continue;
        
        // Record the time to reach this node
        distances.set(node, time);
        
        // Process all neighbors
        for (const { node: neighbor, time: edgeTime } of graph.get(node)) {
            // Only process unvisited neighbors
            if (!distances.has(neighbor)) {
                // Add to queue with cumulative time
                pq.push([neighbor, time + edgeTime]);
            }
        }
    }
    
    // Check if all nodes were reached
    if (distances.size !== n) {
        // Some nodes are unreachable
        return -1;
    }
    
    // Return the maximum time (last node to receive signal)
    return Math.max(...distances.values());
}

// Test cases
console.log(networkDelayTime([[2,1,1],[2,3,1],[3,4,1]], 4, 2)); // 2
console.log(networkDelayTime([[1,2,1]], 2, 1)); // 1
console.log(networkDelayTime([[1,2,1]], 2, 2)); // -1 (can't reach node 1)
console.log(networkDelayTime([[1,2,1],[2,3,2],[1,3,4]], 3, 1)); // 3
```

## 🎨 Step-by-Step Walkthrough

Let's trace through the example: `times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2`

```
Graph structure:
2 → 1 (time: 1)
2 → 3 (time: 1)
3 → 4 (time: 1)

Initial state:
pq = [[2, 0]]
distances = {}

Step 1: Process node 2 at time 0
- Mark node 2 reached at time 0
- Add neighbors: node 1 at time 0+1=1, node 3 at time 0+1=1
- pq = [[1, 1], [3, 1]]
- distances = {2: 0}

Step 2: Process node 1 at time 1
- Mark node 1 reached at time 1
- No neighbors to add
- pq = [[3, 1]]
- distances = {2: 0, 1: 1}

Step 3: Process node 3 at time 1
- Mark node 3 reached at time 1
- Add neighbor: node 4 at time 1+1=2
- pq = [[4, 2]]
- distances = {2: 0, 1: 1, 3: 1}

Step 4: Process node 4 at time 2
- Mark node 4 reached at time 2
- No neighbors to add
- pq = []
- distances = {2: 0, 1: 1, 3: 1, 4: 2}

All nodes reached!
Maximum time = max(0, 1, 1, 2) = 2
```

## 📊 Why This Works

### Dijkstra's Guarantee
```
Dijkstra ensures we find the shortest path to each node:
- Always processes nodes in order of increasing distance
- Once a node is processed, its distance is final
- The maximum distance is when the last node receives the signal
```

### Edge Cases
```
1. Disconnected graph: Some nodes unreachable → return -1
2. Single node: n=1, k=1 → return 0
3. No edges: Only node k is reachable → return -1 if n>1
```

## 🚫 Common Mistakes

1. **Not Handling Unreachable Nodes**
   ```javascript
   // Wrong: Assuming all nodes are reachable
   return Math.max(...distances.values());
   
   // Right: Check if all nodes were reached
   if (distances.size !== n) return -1;
   return Math.max(...distances.values());
   ```

2. **Processing Nodes Multiple Times**
   ```javascript
   // Wrong: Process every entry from priority queue
   const [node, time] = pq.shift();
   distances.set(node, time);  // Might overwrite with larger time!
   
   // Right: Skip already processed nodes
   if (distances.has(node)) continue;
   distances.set(node, time);
   ```

3. **Using 0-Indexed Nodes**
   ```javascript
   // Wrong: Nodes are 1-indexed in this problem
   for (let i = 0; i < n; i++)
   
   // Right: Start from 1
   for (let i = 1; i <= n; i++)
   ```

## 🔄 Alternative Approaches

### Approach 2: Bellman-Ford (Handles Negative Weights)
```javascript
function networkDelayTimeBellmanFord(times, n, k) {
    const distances = new Array(n + 1).fill(Infinity);
    distances[k] = 0;
    
    // Relax edges n-1 times
    for (let i = 0; i < n - 1; i++) {
        for (const [u, v, w] of times) {
            if (distances[u] !== Infinity) {
                distances[v] = Math.min(distances[v], distances[u] + w);
            }
        }
    }
    
    let maxTime = 0;
    for (let i = 1; i <= n; i++) {
        if (distances[i] === Infinity) return -1;
        maxTime = Math.max(maxTime, distances[i]);
    }
    
    return maxTime;
}
// Time: O(V*E), works with negative weights but slower
```

### Approach 3: Floyd-Warshall (All Pairs)
```javascript
function networkDelayTimeFloyd(times, n, k) {
    // Initialize distance matrix
    const dist = Array(n + 1).fill(null)
        .map(() => Array(n + 1).fill(Infinity));
    
    for (let i = 1; i <= n; i++) {
        dist[i][i] = 0;
    }
    
    for (const [u, v, w] of times) {
        dist[u][v] = w;
    }
    
    // Floyd-Warshall
    for (let mid = 1; mid <= n; mid++) {
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= n; j++) {
                dist[i][j] = Math.min(dist[i][j], dist[i][mid] + dist[mid][j]);
            }
        }
    }
    
    let maxTime = 0;
    for (let i = 1; i <= n; i++) {
        if (dist[k][i] === Infinity) return -1;
        maxTime = Math.max(maxTime, dist[k][i]);
    }
    
    return maxTime;
}
// Time: O(V³), overkill for single source but works
```

## 🔗 Related Problems

- **Course Schedule** - Topological sort with prerequisites
- **Cheapest Flights Within K Stops** - Path with constraints
- **Minimum Cost to Connect All Points** - MST problem
- **Path with Maximum Probability** - Modified Dijkstra

## 💡 Key Takeaways

1. **Single-Source Shortest Path**: Classic Dijkstra's application
2. **Maximum of Minimums**: We want the longest of all shortest paths
3. **Handle Disconnected Graphs**: Check if all nodes are reachable
4. **1-Indexed Nodes**: Common in graph problems, be careful!

This problem is perfect for understanding how signals propagate through networks!
