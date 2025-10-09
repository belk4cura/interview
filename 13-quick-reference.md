# Quick Reference Guide for AWS Interviews

## Big O Complexity Cheat Sheet

### Time Complexities
| Algorithm/Operation | Best | Average | Worst | Space |
|-------------------|------|---------|-------|-------|
| **Arrays** |
| Access | O(1) | O(1) | O(1) | - |
| Search | O(1) | O(n) | O(n) | - |
| Insert (end) | O(1) | O(1) | O(1) | - |
| Insert (middle) | O(n) | O(n) | O(n) | - |
| Delete | O(n) | O(n) | O(n) | - |
| **Linked Lists** |
| Access | O(n) | O(n) | O(n) | - |
| Search | O(n) | O(n) | O(n) | - |
| Insert (head) | O(1) | O(1) | O(1) | - |
| Delete (head) | O(1) | O(1) | O(1) | - |
| **Hash Tables** |
| Search | O(1) | O(1) | O(n) | O(n) |
| Insert | O(1) | O(1) | O(n) | O(n) |
| Delete | O(1) | O(1) | O(n) | O(n) |
| **Binary Search Trees** |
| Search | O(log n) | O(log n) | O(n) | O(n) |
| Insert | O(log n) | O(log n) | O(n) | O(n) |
| Delete | O(log n) | O(log n) | O(n) | O(n) |
| **Heaps** |
| Find Min/Max | O(1) | O(1) | O(1) | O(n) |
| Insert | O(log n) | O(log n) | O(log n) | O(n) |
| Delete Min/Max | O(log n) | O(log n) | O(log n) | O(n) |

### Sorting Algorithms
| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+k) | Yes |

### Graph Algorithms
| Algorithm | Time | Space | Use Case |
|-----------|------|-------|----------|
| BFS | O(V+E) | O(V) | Shortest path (unweighted) |
| DFS | O(V+E) | O(V) | Path finding, cycle detection |
| Dijkstra | O((V+E)log V) | O(V) | Shortest path (weighted, positive) |
| Bellman-Ford | O(VE) | O(V) | Shortest path (negative weights) |
| Floyd-Warshall | O(V³) | O(V²) | All pairs shortest path |
| Kruskal's MST | O(E log E) | O(V) | Minimum spanning tree |
| Prim's MST | O((V+E)log V) | O(V) | Minimum spanning tree |
| Topological Sort | O(V+E) | O(V) | Dependency resolution |

## Common Patterns

### Two Pointers
```javascript
// Find pair with target sum
function twoSum(arr, target) {
    let left = 0, right = arr.length - 1;
    while (left < right) {
        const sum = arr[left] + arr[right];
        if (sum === target) return [left, right];
        if (sum < target) left++;
        else right--;
    }
    return [];
}
```

### Sliding Window
```javascript
// Maximum sum subarray of size k
function maxSumSubarray(arr, k) {
    let maxSum = 0, windowSum = 0;
    for (let i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    maxSum = windowSum;
    
    for (let i = k; i < arr.length; i++) {
        windowSum = windowSum - arr[i - k] + arr[i];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

### Fast & Slow Pointers
```javascript
// Detect cycle in linked list
function hasCycle(head) {
    let slow = head, fast = head;
    while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow === fast) return true;
    }
    return false;
}
```

### Merge Intervals
```javascript
// Merge overlapping intervals
function mergeIntervals(intervals) {
    intervals.sort((a, b) => a[0] - b[0]);
    const merged = [intervals[0]];
    
    for (let i = 1; i < intervals.length; i++) {
        const last = merged[merged.length - 1];
        if (intervals[i][0] <= last[1]) {
            last[1] = Math.max(last[1], intervals[i][1]);
        } else {
            merged.push(intervals[i]);
        }
    }
    return merged;
}
```

### Binary Search Template
```javascript
// Standard binary search
function binarySearch(arr, target) {
    let left = 0, right = arr.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        if (arr[mid] === target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

### BFS Template
```javascript
// Level-order traversal
function bfs(root) {
    if (!root) return [];
    const queue = [root];
    const result = [];
    
    while (queue.length) {
        const levelSize = queue.length;
        const level = [];
        
        for (let i = 0; i < levelSize; i++) {
            const node = queue.shift();
            level.push(node.val);
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        result.push(level);
    }
    return result;
}
```

### DFS Template
```javascript
// Tree traversal
function dfs(root) {
    const result = [];
    
    function traverse(node) {
        if (!node) return;
        result.push(node.val);     // Preorder
        traverse(node.left);
        // result.push(node.val);   // Inorder
        traverse(node.right);
        // result.push(node.val);   // Postorder
    }
    
    traverse(root);
    return result;
}
```

### Dynamic Programming Template
```javascript
// Fibonacci with memoization
function fib(n, memo = {}) {
    if (n in memo) return memo[n];
    if (n <= 1) return n;
    
    memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
    return memo[n];
}

// Tabulation approach
function fibTab(n) {
    if (n <= 1) return n;
    const dp = [0, 1];
    
    for (let i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

## AWS Services Quick Reference

### Compute
| Service | Use Case | Key Features |
|---------|----------|--------------|
| EC2 | Virtual servers | Auto-scaling, spot instances |
| Lambda | Serverless functions | Event-driven, 15 min max |
| ECS | Container orchestration | Docker, Fargate |
| EKS | Kubernetes | Managed K8s |
| Batch | Batch computing | Job queues, scheduling |

### Storage
| Service | Use Case | Key Features |
|---------|----------|--------------|
| S3 | Object storage | 99.999999999% durability |
| EBS | Block storage | EC2 volumes, snapshots |
| EFS | File storage | NFSv4, multi-AZ |
| Glacier | Archive storage | Low cost, retrieval time |
| Storage Gateway | Hybrid storage | On-premise integration |

### Database
| Service | Type | Use Case |
|---------|------|----------|
| RDS | Relational | ACID, SQL |
| DynamoDB | NoSQL | Key-value, document |
| ElastiCache | In-memory | Redis, Memcached |
| DocumentDB | Document | MongoDB compatible |
| Neptune | Graph | Social networks, recommendations |
| Timestream | Time-series | IoT, metrics |
| Redshift | Data warehouse | Analytics, OLAP |

### Networking
| Service | Purpose |
|---------|---------|
| VPC | Virtual network |
| CloudFront | CDN |
| Route 53 | DNS |
| API Gateway | API management |
| Direct Connect | Dedicated connection |
| Transit Gateway | Network hub |
| Global Accelerator | Global routing |

### Analytics
| Service | Use Case |
|---------|----------|
| Kinesis | Real-time streaming |
| EMR | Big data processing |
| Athena | S3 SQL queries |
| QuickSight | BI visualization |
| Glue | ETL |
| Lake Formation | Data lake |
| MSK | Managed Kafka |

## System Design Estimates

### Capacity Planning
```
1 Million users
- Read QPS: 1M * 10 requests/day / 86400 ≈ 120 QPS
- Write QPS: 1M * 1 write/day / 86400 ≈ 12 QPS
- Peak QPS: Average * 3-5x

Storage:
- Text post: ~1KB
- Image: ~200KB compressed
- Video: ~50MB per minute

Bandwidth:
- 100 QPS * 1MB response = 100MB/s = 800 Mbps

Database:
- 1M users * 1KB = 1GB user data
- 1M posts/day * 1KB * 365 = 365GB/year
```

### Latency Numbers
```
L1 cache: 0.5 ns
L2 cache: 7 ns
RAM: 100 ns
SSD: 150 μs
HDD: 10 ms
Network (same region): 0.5 ms
Network (cross-region): 50-150 ms
Network (cross-continent): 150-300 ms
```

### AWS Limits
```
Lambda: 15 min timeout, 10GB memory
API Gateway: 29 sec timeout, 10MB payload
S3: 5TB object size, 5GB single PUT
DynamoDB: 400KB item size, 40K RCU/WCU per table
SQS: 256KB message, 14 days retention
EC2: 500 instances default (soft limit)
```

## SQL vs NoSQL Decision Tree

```
Need ACID transactions? → SQL
Need complex queries/joins? → SQL
Need flexible schema? → NoSQL
Need horizontal scaling? → NoSQL
Need sub-millisecond latency? → NoSQL (in-memory)
Need full-text search? → Specialized (ElasticSearch)
Need graph relationships? → Graph DB
Need time-series data? → Time-series DB
```

## Scalability Checklist

- [ ] **Vertical Scaling**: Increase instance size
- [ ] **Horizontal Scaling**: Add more instances
- [ ] **Load Balancing**: Distribute traffic (ALB, NLB)
- [ ] **Caching**: CloudFront, ElastiCache, application cache
- [ ] **Database Optimization**: Indexes, read replicas, sharding
- [ ] **Async Processing**: SQS, SNS, EventBridge
- [ ] **Auto-scaling**: Target tracking, scheduled scaling
- [ ] **CDN**: Static content, edge locations
- [ ] **Microservices**: Decouple components
- [ ] **Data Partitioning**: Distribute data across nodes

## Interview Day Checklist

### Before Interview
- [ ] Review AWS Leadership Principles
- [ ] Prepare 2-3 STAR stories per principle
- [ ] Review common algorithms and data structures
- [ ] Practice system design on whiteboard
- [ ] Prepare questions to ask interviewer

### During Interview
- [ ] Clarify requirements before solving
- [ ] Think out loud
- [ ] Consider edge cases
- [ ] Analyze time/space complexity
- [ ] Test your solution with examples
- [ ] Optimize if time permits

### System Design Approach
1. **Requirements** (5 min)
   - Functional requirements
   - Non-functional requirements
   - Constraints

2. **Estimation** (5 min)
   - Traffic (QPS)
   - Storage
   - Bandwidth

3. **High-level Design** (10 min)
   - Architecture diagram
   - Major components
   - Data flow

4. **Detailed Design** (20 min)
   - Database schema
   - API design
   - Algorithm selection

5. **Scale & Optimize** (10 min)
   - Bottlenecks
   - Caching strategy
   - Cost optimization

## Common Mistakes to Avoid

### Coding
- ❌ Not checking null/empty inputs
- ❌ Off-by-one errors in loops
- ❌ Forgetting to handle edge cases
- ❌ Not considering integer overflow
- ❌ Modifying collection while iterating

### System Design
- ❌ Over-engineering for initial requirements
- ❌ Ignoring data consistency
- ❌ Not considering failure modes
- ❌ Forgetting about monitoring/logging
- ❌ Neglecting cost optimization

### Behavioral
- ❌ Using "we" instead of "I"
- ❌ Not quantifying impact
- ❌ Blaming others for failures
- ❌ Generic answers without specifics
- ❌ Not showing growth/learning

## Quick Formulas

```javascript
// Combinations: n choose k
C(n,k) = n! / (k! * (n-k)!)

// Permutations: n pick k with order
P(n,k) = n! / (n-k)!

// Sum of arithmetic sequence
Sum = n * (first + last) / 2

// Sum of geometric sequence
Sum = a * (1 - r^n) / (1 - r)

// Master Theorem: T(n) = aT(n/b) + f(n)
If f(n) = O(n^c) where c < log_b(a): T(n) = O(n^log_b(a))
If f(n) = O(n^c) where c = log_b(a): T(n) = O(n^c * log n)
If f(n) = O(n^c) where c > log_b(a): T(n) = O(f(n))
```

---

*Previous: [← System Design Examples](./12-system-design-examples.md)*
*[Back to Main README →](./README.md)*
