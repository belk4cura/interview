# Scalability Patterns

## Overview
Scalability patterns are architectural approaches that enable systems to handle increased load gracefully. These patterns are fundamental to building robust AWS applications that can grow from handling hundreds to millions of requests.

### Key Concepts
- **Horizontal vs Vertical Scaling**: Scale out vs scale up
- **Load Balancing**: Distributing traffic across resources
- **Partitioning**: Dividing data for parallel processing
- **Redundancy**: Eliminating single points of failure
- **AWS Applications**: Auto Scaling, ELB, Multi-AZ deployments

## Scaling Strategies

### Vertical Scaling (Scale Up)
```javascript
/**
 * Vertical scaling simulation
 * Upgrade instance resources
 */
class VerticalScalingManager {
    constructor() {
        this.instanceTypes = [
            { type: 't2.micro', cpu: 1, memory: 1, cost: 0.0116 },
            { type: 't2.small', cpu: 1, memory: 2, cost: 0.023 },
            { type: 't2.medium', cpu: 2, memory: 4, cost: 0.0464 },
            { type: 't2.large', cpu: 2, memory: 8, cost: 0.0928 },
            { type: 't2.xlarge', cpu: 4, memory: 16, cost: 0.1856 },
            { type: 't2.2xlarge', cpu: 8, memory: 32, cost: 0.3712 }
        ];
        this.currentInstance = 0;
    }
    
    getCurrentCapacity() {
        return this.instanceTypes[this.currentInstance];
    }
    
    canScaleUp() {
        return this.currentInstance < this.instanceTypes.length - 1;
    }
    
    scaleUp() {
        if (!this.canScaleUp()) {
            throw new Error('Already at maximum capacity');
        }
        
        const oldInstance = this.instanceTypes[this.currentInstance];
        this.currentInstance++;
        const newInstance = this.instanceTypes[this.currentInstance];
        
        return {
            upgraded: true,
            from: oldInstance,
            to: newInstance,
            downtime: '5-10 minutes', // Typical EC2 resize time
            costIncrease: ((newInstance.cost - oldInstance.cost) / oldInstance.cost * 100).toFixed(2) + '%'
        };
    }
    
    // Calculate if scaling is needed based on metrics
    shouldScale(cpuUtilization, memoryUtilization) {
        const threshold = 80; // 80% utilization threshold
        return cpuUtilization > threshold || memoryUtilization > threshold;
    }
}
```

### Horizontal Scaling (Scale Out)
```javascript
/**
 * Horizontal scaling with load distribution
 * Add/remove instances based on load
 */
class HorizontalScalingManager {
    constructor(config = {}) {
        this.minInstances = config.minInstances || 2;
        this.maxInstances = config.maxInstances || 10;
        this.targetUtilization = config.targetUtilization || 70;
        this.instances = [];
        this.scaleUpThreshold = 80;
        this.scaleDownThreshold = 30;
        
        // Initialize minimum instances
        for (let i = 0; i < this.minInstances; i++) {
            this.addInstance();
        }
    }
    
    addInstance() {
        if (this.instances.length >= this.maxInstances) {
            return { scaled: false, reason: 'Max capacity reached' };
        }
        
        const instance = {
            id: `i-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
            capacity: 100,
            currentLoad: 0,
            health: 'healthy',
            launchedAt: Date.now()
        };
        
        this.instances.push(instance);
        
        return {
            scaled: true,
            instance,
            totalInstances: this.instances.length
        };
    }
    
    removeInstance() {
        if (this.instances.length <= this.minInstances) {
            return { scaled: false, reason: 'Min capacity reached' };
        }
        
        // Remove instance with least connections (graceful)
        const instanceToRemove = this.instances
            .sort((a, b) => a.currentLoad - b.currentLoad)[0];
        
        this.instances = this.instances.filter(i => i.id !== instanceToRemove.id);
        
        return {
            scaled: true,
            removed: instanceToRemove.id,
            totalInstances: this.instances.length
        };
    }
    
    // Auto-scaling based on metrics
    autoScale(avgCpuUtilization) {
        const decisions = [];
        
        if (avgCpuUtilization > this.scaleUpThreshold) {
            // Scale up
            const instancesToAdd = Math.ceil(
                (avgCpuUtilization - this.targetUtilization) / 10
            );
            
            for (let i = 0; i < instancesToAdd; i++) {
                const result = this.addInstance();
                if (result.scaled) {
                    decisions.push({
                        action: 'scale_up',
                        reason: `CPU utilization ${avgCpuUtilization}% > ${this.scaleUpThreshold}%`,
                        ...result
                    });
                }
            }
        } else if (avgCpuUtilization < this.scaleDownThreshold) {
            // Scale down
            const instancesToRemove = Math.floor(
                (this.targetUtilization - avgCpuUtilization) / 20
            );
            
            for (let i = 0; i < instancesToRemove; i++) {
                const result = this.removeInstance();
                if (result.scaled) {
                    decisions.push({
                        action: 'scale_down',
                        reason: `CPU utilization ${avgCpuUtilization}% < ${this.scaleDownThreshold}%`,
                        ...result
                    });
                }
            }
        }
        
        return decisions;
    }
    
    // Distribute load across instances
    distributeLoad(totalRequests) {
        const requestsPerInstance = Math.ceil(totalRequests / this.instances.length);
        
        this.instances.forEach(instance => {
            instance.currentLoad = Math.min(requestsPerInstance, instance.capacity);
        });
        
        return {
            totalCapacity: this.instances.reduce((sum, i) => sum + i.capacity, 0),
            totalLoad: totalRequests,
            distribution: this.instances.map(i => ({
                id: i.id,
                load: i.currentLoad,
                utilization: (i.currentLoad / i.capacity * 100).toFixed(2) + '%'
            }))
        };
    }
}
```

## Load Balancing

### Round Robin Load Balancer
```javascript
/**
 * Simple round-robin load distribution
 */
class RoundRobinLoadBalancer {
    constructor(servers) {
        this.servers = servers;
        this.currentIndex = 0;
    }
    
    getNextServer() {
        const server = this.servers[this.currentIndex];
        this.currentIndex = (this.currentIndex + 1) % this.servers.length;
        return server;
    }
    
    handleRequest(request) {
        const server = this.getNextServer();
        return {
            server: server.id,
            response: server.processRequest(request)
        };
    }
}
```

### Weighted Load Balancer
```javascript
/**
 * Weighted distribution based on server capacity
 */
class WeightedLoadBalancer {
    constructor(servers) {
        this.servers = servers;
        this.weightedList = this.buildWeightedList();
        this.currentIndex = 0;
    }
    
    buildWeightedList() {
        const list = [];
        for (const server of this.servers) {
            for (let i = 0; i < server.weight; i++) {
                list.push(server);
            }
        }
        return list;
    }
    
    getNextServer() {
        const server = this.weightedList[this.currentIndex];
        this.currentIndex = (this.currentIndex + 1) % this.weightedList.length;
        return server;
    }
}
```

### Least Connections Load Balancer
```javascript
/**
 * Route to server with fewest active connections
 */
class LeastConnectionsLoadBalancer {
    constructor(servers) {
        this.servers = servers.map(s => ({
            ...s,
            activeConnections: 0
        }));
    }
    
    getNextServer() {
        // Find server with least connections
        return this.servers.reduce((min, server) => 
            server.activeConnections < min.activeConnections ? server : min
        );
    }
    
    handleRequest(request) {
        const server = this.getNextServer();
        server.activeConnections++;
        
        // Simulate request processing
        setTimeout(() => {
            server.activeConnections--;
        }, request.processingTime || 1000);
        
        return {
            server: server.id,
            connections: server.activeConnections
        };
    }
}
```

### Consistent Hash Load Balancer
```javascript
/**
 * Consistent hashing for sticky sessions
 */
class ConsistentHashLoadBalancer {
    constructor(servers, virtualNodes = 150) {
        this.servers = new Map();
        this.ring = new Map();
        this.virtualNodes = virtualNodes;
        
        for (const server of servers) {
            this.addServer(server);
        }
    }
    
    hash(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
            hash = hash & hash;
        }
        return Math.abs(hash);
    }
    
    addServer(server) {
        this.servers.set(server.id, server);
        
        // Add virtual nodes for better distribution
        for (let i = 0; i < this.virtualNodes; i++) {
            const virtualKey = `${server.id}:${i}`;
            const hash = this.hash(virtualKey);
            this.ring.set(hash, server);
        }
    }
    
    removeServer(serverId) {
        if (!this.servers.has(serverId)) return;
        
        const server = this.servers.get(serverId);
        
        // Remove virtual nodes
        for (let i = 0; i < this.virtualNodes; i++) {
            const virtualKey = `${serverId}:${i}`;
            const hash = this.hash(virtualKey);
            this.ring.delete(hash);
        }
        
        this.servers.delete(serverId);
    }
    
    getServer(sessionId) {
        if (this.ring.size === 0) return null;
        
        const hash = this.hash(sessionId);
        const sortedHashes = Array.from(this.ring.keys()).sort((a, b) => a - b);
        
        // Find first hash >= session hash
        let selectedHash = sortedHashes[0];
        for (const h of sortedHashes) {
            if (h >= hash) {
                selectedHash = h;
                break;
            }
        }
        
        return this.ring.get(selectedHash);
    }
}
```

## Data Partitioning

### Range Partitioning
```javascript
/**
 * Partition data by range
 */
class RangePartitioner {
    constructor(partitions) {
        this.partitions = partitions; // [{min: 0, max: 100, node: 'node1'}, ...]
    }
    
    getPartition(key) {
        for (const partition of this.partitions) {
            if (key >= partition.min && key < partition.max) {
                return partition.node;
            }
        }
        throw new Error(`No partition found for key: ${key}`);
    }
    
    // Rebalance partitions when adding nodes
    rebalance(newNodeCount) {
        const totalRange = this.partitions[this.partitions.length - 1].max;
        const rangePerNode = Math.ceil(totalRange / newNodeCount);
        
        const newPartitions = [];
        for (let i = 0; i < newNodeCount; i++) {
            newPartitions.push({
                min: i * rangePerNode,
                max: Math.min((i + 1) * rangePerNode, totalRange),
                node: `node${i + 1}`
            });
        }
        
        // Migration plan
        const migrations = this.calculateMigrations(this.partitions, newPartitions);
        
        this.partitions = newPartitions;
        
        return {
            newPartitions,
            migrations
        };
    }
    
    calculateMigrations(oldPartitions, newPartitions) {
        const migrations = [];
        
        for (const oldPart of oldPartitions) {
            for (const newPart of newPartitions) {
                if (oldPart.node !== newPart.node) {
                    const overlapMin = Math.max(oldPart.min, newPart.min);
                    const overlapMax = Math.min(oldPart.max, newPart.max);
                    
                    if (overlapMin < overlapMax) {
                        migrations.push({
                            range: [overlapMin, overlapMax],
                            from: oldPart.node,
                            to: newPart.node
                        });
                    }
                }
            }
        }
        
        return migrations;
    }
}
```

### Hash Partitioning
```javascript
/**
 * Partition data by hash
 */
class HashPartitioner {
    constructor(nodeCount) {
        this.nodeCount = nodeCount;
        this.nodes = Array.from({ length: nodeCount }, (_, i) => `node${i + 1}`);
    }
    
    hash(key) {
        let hash = 0;
        for (let i = 0; i < key.toString().length; i++) {
            hash = ((hash << 5) - hash) + key.toString().charCodeAt(i);
            hash = hash & hash;
        }
        return Math.abs(hash);
    }
    
    getPartition(key) {
        const hash = this.hash(key);
        const partition = hash % this.nodeCount;
        return this.nodes[partition];
    }
    
    // Consistent hashing for better rebalancing
    consistentHash(key, virtualNodes = 150) {
        // Implementation similar to ConsistentHashLoadBalancer
        return this.getPartition(key);
    }
}
```

### Composite Partitioning
```javascript
/**
 * Multi-level partitioning strategy
 */
class CompositePartitioner {
    constructor() {
        this.levels = [];
    }
    
    addLevel(strategy, config) {
        this.levels.push({ strategy, config });
    }
    
    getPartition(key) {
        let currentPartition = 'root';
        
        for (const level of this.levels) {
            currentPartition = this.applyStrategy(
                key,
                level.strategy,
                level.config,
                currentPartition
            );
        }
        
        return currentPartition;
    }
    
    applyStrategy(key, strategy, config, parentPartition) {
        switch (strategy) {
            case 'range':
                return `${parentPartition}-range-${this.getRangePartition(key, config)}`;
            case 'hash':
                return `${parentPartition}-hash-${this.getHashPartition(key, config)}`;
            case 'list':
                return `${parentPartition}-list-${this.getListPartition(key, config)}`;
            default:
                return parentPartition;
        }
    }
    
    getRangePartition(key, config) {
        for (const [name, range] of Object.entries(config.ranges)) {
            if (key >= range[0] && key < range[1]) {
                return name;
            }
        }
        return 'default';
    }
    
    getHashPartition(key, config) {
        const hash = this.hash(key);
        return hash % config.buckets;
    }
    
    getListPartition(key, config) {
        return config.mapping[key] || 'default';
    }
    
    hash(key) {
        let hash = 0;
        for (let i = 0; i < key.toString().length; i++) {
            hash = ((hash << 5) - hash) + key.toString().charCodeAt(i);
        }
        return Math.abs(hash);
    }
}
```

## Redundancy and Fault Tolerance

### Active-Passive Redundancy
```javascript
/**
 * Primary-backup configuration
 */
class ActivePassiveSystem {
    constructor() {
        this.primary = {
            id: 'primary',
            status: 'active',
            health: 100,
            lastHeartbeat: Date.now()
        };
        
        this.backup = {
            id: 'backup',
            status: 'standby',
            health: 100,
            lastHeartbeat: Date.now()
        };
        
        this.heartbeatInterval = 5000; // 5 seconds
        this.failoverThreshold = 3; // missed heartbeats
        this.missedHeartbeats = 0;
    }
    
    checkHealth() {
        const now = Date.now();
        const timeSinceLastHeartbeat = now - this.primary.lastHeartbeat;
        
        if (timeSinceLastHeartbeat > this.heartbeatInterval) {
            this.missedHeartbeats++;
            
            if (this.missedHeartbeats >= this.failoverThreshold) {
                return this.failover();
            }
        } else {
            this.missedHeartbeats = 0;
        }
        
        return {
            status: 'healthy',
            primary: this.primary.id,
            backup: this.backup.id
        };
    }
    
    failover() {
        // Swap primary and backup
        const temp = this.primary;
        this.primary = this.backup;
        this.backup = temp;
        
        this.primary.status = 'active';
        this.backup.status = 'failed';
        
        return {
            status: 'failover',
            newPrimary: this.primary.id,
            failedNode: this.backup.id,
            timestamp: Date.now()
        };
    }
    
    recover() {
        if (this.backup.status === 'failed') {
            this.backup.status = 'standby';
            this.backup.health = 100;
            this.backup.lastHeartbeat = Date.now();
            
            return {
                status: 'recovered',
                node: this.backup.id
            };
        }
    }
}
```

### Active-Active Redundancy
```javascript
/**
 * Multiple active nodes with load distribution
 */
class ActiveActiveSystem {
    constructor(nodeCount = 3) {
        this.nodes = Array.from({ length: nodeCount }, (_, i) => ({
            id: `node${i + 1}`,
            status: 'active',
            health: 100,
            load: 0,
            lastHeartbeat: Date.now()
        }));
        
        this.quorum = Math.floor(nodeCount / 2) + 1;
    }
    
    handleRequest(request) {
        const healthyNodes = this.nodes.filter(n => n.status === 'active');
        
        if (healthyNodes.length < this.quorum) {
            throw new Error('Insufficient healthy nodes for quorum');
        }
        
        // Round-robin among healthy nodes
        const node = healthyNodes[Math.floor(Math.random() * healthyNodes.length)];
        node.load++;
        
        return {
            handledBy: node.id,
            quorum: `${healthyNodes.length}/${this.nodes.length}`,
            result: this.processRequest(request)
        };
    }
    
    processRequest(request) {
        // Simulate processing
        return {
            success: true,
            timestamp: Date.now(),
            data: request
        };
    }
    
    detectFailure(nodeId) {
        const node = this.nodes.find(n => n.id === nodeId);
        if (node) {
            node.status = 'failed';
            
            // Check if we still have quorum
            const healthyNodes = this.nodes.filter(n => n.status === 'active');
            
            return {
                failedNode: nodeId,
                remainingNodes: healthyNodes.length,
                hasQuorum: healthyNodes.length >= this.quorum
            };
        }
    }
    
    addNode() {
        const newNode = {
            id: `node${this.nodes.length + 1}`,
            status: 'syncing',
            health: 100,
            load: 0,
            lastHeartbeat: Date.now()
        };
        
        this.nodes.push(newNode);
        
        // Simulate sync time
        setTimeout(() => {
            newNode.status = 'active';
        }, 5000);
        
        return {
            added: newNode.id,
            totalNodes: this.nodes.length,
            newQuorum: Math.floor(this.nodes.length / 2) + 1
        };
    }
}
```

## AWS Implementation Examples

### Auto Scaling Group Configuration
```javascript
/**
 * AWS Auto Scaling Group simulation
 */
class AutoScalingGroup {
    constructor(config) {
        this.name = config.name;
        this.minSize = config.minSize || 1;
        this.maxSize = config.maxSize || 10;
        this.desiredCapacity = config.desiredCapacity || this.minSize;
        this.healthCheckGracePeriod = config.healthCheckGracePeriod || 300;
        this.cooldown = config.cooldown || 300;
        
        this.instances = [];
        this.policies = [];
        this.lastScalingActivity = 0;
        
        // Launch initial instances
        this.launchInstances(this.desiredCapacity);
    }
    
    launchInstances(count) {
        for (let i = 0; i < count; i++) {
            if (this.instances.length >= this.maxSize) break;
            
            const instance = {
                id: `i-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
                state: 'pending',
                health: 'healthy',
                launchTime: Date.now(),
                metrics: {
                    cpu: 0,
                    memory: 0,
                    network: 0
                }
            };
            
            this.instances.push(instance);
            
            // Simulate launch time
            setTimeout(() => {
                instance.state = 'running';
            }, 30000);
        }
        
        this.lastScalingActivity = Date.now();
    }
    
    terminateInstances(count) {
        for (let i = 0; i < count; i++) {
            if (this.instances.length <= this.minSize) break;
            
            // Terminate oldest instance first
            const instance = this.instances
                .filter(i => i.state === 'running')
                .sort((a, b) => a.launchTime - b.launchTime)[0];
            
            if (instance) {
                instance.state = 'terminating';
                
                setTimeout(() => {
                    this.instances = this.instances.filter(i => i.id !== instance.id);
                }, 10000);
            }
        }
        
        this.lastScalingActivity = Date.now();
    }
    
    addScalingPolicy(policy) {
        this.policies.push(policy);
    }
    
    evaluateScalingPolicies(metrics) {
        // Check cooldown period
        if (Date.now() - this.lastScalingActivity < this.cooldown * 1000) {
            return { scaled: false, reason: 'In cooldown period' };
        }
        
        for (const policy of this.policies) {
            const shouldScale = this.evaluatePolicy(policy, metrics);
            
            if (shouldScale) {
                return this.executePolicy(policy);
            }
        }
        
        return { scaled: false };
    }
    
    evaluatePolicy(policy, metrics) {
        switch (policy.metricType) {
            case 'CPUUtilization':
                return this.evaluateMetric(
                    metrics.avgCpu,
                    policy.threshold,
                    policy.comparisonOperator
                );
            case 'RequestCount':
                return this.evaluateMetric(
                    metrics.requestCount,
                    policy.threshold,
                    policy.comparisonOperator
                );
            default:
                return false;
        }
    }
    
    evaluateMetric(value, threshold, operator) {
        switch (operator) {
            case 'GreaterThanThreshold':
                return value > threshold;
            case 'LessThanThreshold':
                return value < threshold;
            default:
                return false;
        }
    }
    
    executePolicy(policy) {
        const currentSize = this.instances.filter(i => 
            i.state === 'running' || i.state === 'pending'
        ).length;
        
        let adjustment = 0;
        
        switch (policy.adjustmentType) {
            case 'ChangeInCapacity':
                adjustment = policy.scalingAdjustment;
                break;
            case 'PercentChangeInCapacity':
                adjustment = Math.ceil(currentSize * policy.scalingAdjustment / 100);
                break;
            case 'ExactCapacity':
                adjustment = policy.scalingAdjustment - currentSize;
                break;
        }
        
        if (adjustment > 0) {
            this.launchInstances(adjustment);
            return {
                scaled: true,
                action: 'scale_out',
                instances: adjustment,
                policy: policy.name
            };
        } else if (adjustment < 0) {
            this.terminateInstances(Math.abs(adjustment));
            return {
                scaled: true,
                action: 'scale_in',
                instances: Math.abs(adjustment),
                policy: policy.name
            };
        }
        
        return { scaled: false };
    }
}

// Usage
const asg = new AutoScalingGroup({
    name: 'web-servers',
    minSize: 2,
    maxSize: 10,
    desiredCapacity: 3
});

asg.addScalingPolicy({
    name: 'scale-out-high-cpu',
    metricType: 'CPUUtilization',
    threshold: 70,
    comparisonOperator: 'GreaterThanThreshold',
    adjustmentType: 'ChangeInCapacity',
    scalingAdjustment: 2
});

asg.addScalingPolicy({
    name: 'scale-in-low-cpu',
    metricType: 'CPUUtilization',
    threshold: 30,
    comparisonOperator: 'LessThanThreshold',
    adjustmentType: 'ChangeInCapacity',
    scalingAdjustment: -1
});
```

## Interview Tips

### Common Questions
1. **"Horizontal vs Vertical Scaling?"**
   - Horizontal: Add more machines, better fault tolerance
   - Vertical: Upgrade machine resources, has limits
   - Horizontal preferred for cloud-native apps

2. **"How to handle stateful services?"**
   - Sticky sessions with consistent hashing
   - External state stores (Redis, DynamoDB)
   - State replication across nodes

3. **"Explain CAP theorem"**
   - Consistency, Availability, Partition tolerance
   - Can only guarantee 2 out of 3
   - Trade-offs based on use case

### Red Flags to Avoid
- ❌ Not considering data consistency in scaling
- ❌ Ignoring network partitions
- ❌ Over-engineering for unlikely scale
- ❌ Not planning for graceful degradation

### AWS-Specific Follow-ups
- "Design multi-region active-active setup"
- "Implement blue-green deployment strategy"
- "Optimize Auto Scaling for cost"
- "Handle AZ failures gracefully"

## Key Takeaways
1. **Horizontal scaling** provides better fault tolerance
2. **Load balancing** distributes traffic effectively
3. **Partitioning** enables parallel processing
4. **Redundancy** eliminates single points of failure
5. **Auto Scaling** responds to demand automatically

---

*Previous: [← Graph Algorithms](./07-graph-algorithms.md)*
*Next: [Caching and Performance →](./09-caching-and-performance.md)*
