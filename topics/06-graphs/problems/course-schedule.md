# Course Schedule

## 🎯 What Are We Trying To Solve?

Determining if you can complete all courses given prerequisites is like planning your college schedule - some courses require others first, and we need to detect if there's an impossible circular dependency.

## 📝 The Problem

Given n courses and prerequisites, determine if it's possible to finish all courses (detect cycles in directed graph).

```
Example:
numCourses = 4
prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: true (possible order: 0, 1, 2, 3)

numCourses = 2
prerequisites = [[1,0],[0,1]]
Output: false (circular dependency)
```

## 💻 The Code

```javascript
/**
 * Course Schedule - DFS Cycle Detection
 * Time: O(V + E), Space: O(V)
 */
function canFinish(numCourses, prerequisites) {
    // Build adjacency list
    const graph = Array.from({length: numCourses}, () => []);
    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);
    }
    
    // 0: unvisited, 1: visiting, 2: visited
    const state = new Array(numCourses).fill(0);
    
    function hasCycle(course) {
        if (state[course] === 1) return true;  // Found cycle
        if (state[course] === 2) return false; // Already processed
        
        state[course] = 1; // Mark as visiting
        
        for (const next of graph[course]) {
            if (hasCycle(next)) return true;
        }
        
        state[course] = 2; // Mark as visited
        return false;
    }
    
    // Check all courses
    for (let i = 0; i < numCourses; i++) {
        if (hasCycle(i)) return false;
    }
    
    return true;
}

/**
 * Course Schedule II - Return order (Topological Sort)
 * Time: O(V + E), Space: O(V)
 */
function findOrder(numCourses, prerequisites) {
    const graph = Array.from({length: numCourses}, () => []);
    const indegree = new Array(numCourses).fill(0);
    
    // Build graph and count indegrees
    for (const [course, prereq] of prerequisites) {
        graph[prereq].push(course);
        indegree[course]++;
    }
    
    // BFS with queue
    const queue = [];
    const result = [];
    
    // Add all courses with no prerequisites
    for (let i = 0; i < numCourses; i++) {
        if (indegree[i] === 0) queue.push(i);
    }
    
    while (queue.length) {
        const course = queue.shift();
        result.push(course);
        
        // Process dependent courses
        for (const next of graph[course]) {
            indegree[next]--;
            if (indegree[next] === 0) {
                queue.push(next);
            }
        }
    }
    
    return result.length === numCourses ? result : [];
}
```

## 🎨 Key Insights

- **Cycle Detection**: Use DFS with three states
- **Topological Sort**: BFS/Kahn's algorithm or DFS
- **Indegree Counting**: Track prerequisites remaining

## ⏱️ Complexity

- Time: O(V + E) where V = courses, E = prerequisites
- Space: O(V) for visited array and recursion stack
