# Distributed Systems

## Overview
Distributed systems are collections of independent computers that appear as a single coherent system. Understanding distributed computing principles is crucial for designing resilient, scalable AWS architectures.

### Key Concepts
- **CAP Theorem**: Consistency, Availability, Partition tolerance
- **Consensus Algorithms**: Raft, Paxos, Byzantine fault tolerance
- **Distributed Coordination**: Leader election, distributed locks
- **AWS Applications**: Multi-region deployments, microservices, serverless

## CAP Theorem and Trade-offs

### Understanding CAP
```javascript
/**
 * CAP Theorem Implementation Examples
 * Can only guarantee 2 out of 3: Consistency, Availability, Partition Tolerance
 */

// CP System - Sacrifices Availability for Consistency
class CPSystem {
    constructor() {
        this.nodes = new Map();
        this.partitioned = new Set();
        this.quorum = 0.51; // Majority quorum
    }
    
    write(key, value) {
        const availableNodes = this.getAvailableNodes();
        const requiredNodes = Math.ceil(this.nodes.size * this.quorum);
        
        if (availableNodes.length < requiredNodes) {
            throw new Error('Insufficient nodes for quorum - write rejected');
        }
        
        // Write to all available nodes (strong consistency)
        const promises = availableNodes.map(node => 
            node.write(key, value)
        );
        
        return Promise.all(promises);
    }
    
    read(key) {
        const availableNodes = this.getAvailableNodes();
        const requiredNodes = Math.ceil(this.nodes.size * this.quorum);
        
        if (availableNodes.length < requiredNodes) {
            throw new Error('Insufficient nodes for quorum - read rejected');
        }
        
        // Read from quorum and return latest
        return this.readWithQuorum(key, availableNodes);
    }
    
    getAvailableNodes() {
        return Array.from(this.nodes.values())
            .filter(node => !this.partitioned.has(node.id));
    }
}

// AP System - Sacrifices Consistency for Availability
class APSystem {
    constructor() {
        this.nodes = new Map();
        this.partitioned = new Set();
    }
    
    async write(key, value) {
        const availableNodes = this.getAvailableNodes();
        
        if (availableNodes.length === 0) {
            throw new Error('No nodes available');
        }
        
        // Write to any available node (eventual consistency)
        const promises = availableNodes.map(node => 
            node.write(key, value).catch(() => null)
        );
        
        const results = await Promise.allSettled(promises);
        const succeeded = results.filter(r => r.status === 'fulfilled');
        
        if (succeeded.length > 0) {
            return true; // At least one write succeeded
        }
        
        throw new Error('All writes failed');
    }
    
    async read(key) {
        const availableNodes = this.getAvailableNodes();
        
        // Read from any available node
        for (const node of availableNodes) {
            try {
                return await node.read(key);
            } catch (error) {
                continue; // Try next node
            }
        }
        
        throw new Error('No nodes could serve the read');
    }
    
    getAvailableNodes() {
        return Array.from(this.nodes.values())
            .filter(node => !this.partitioned.has(node.id));
    }
}
```

## Consensus Algorithms

### Raft Implementation
```javascript
/**
 * Simplified Raft consensus algorithm
 */
class RaftNode {
    constructor(id, peers) {
        this.id = id;
        this.peers = peers;
        this.state = 'follower'; // follower, candidate, leader
        this.currentTerm = 0;
        this.votedFor = null;
        this.log = [];
        this.commitIndex = 0;
        this.lastApplied = 0;
        
        // Leader only
        this.nextIndex = new Map();
        this.matchIndex = new Map();
        
        // Start election timer
        this.resetElectionTimer();
    }
    
    // Request vote for leader election
    requestVote(term, candidateId, lastLogIndex, lastLogTerm) {
        if (term < this.currentTerm) {
            return { term: this.currentTerm, voteGranted: false };
        }
        
        if (term > this.currentTerm) {
            this.currentTerm = term;
            this.votedFor = null;
            this.state = 'follower';
        }
        
        const logOk = this.isLogUpToDate(lastLogIndex, lastLogTerm);
        
        if ((this.votedFor === null || this.votedFor === candidateId) && logOk) {
            this.votedFor = candidateId;
            this.resetElectionTimer();
            return { term: this.currentTerm, voteGranted: true };
        }
        
        return { term: this.currentTerm, voteGranted: false };
    }
    
    // Append entries (heartbeat/replication)
    appendEntries(term, leaderId, prevLogIndex, prevLogTerm, entries, leaderCommit) {
        if (term < this.currentTerm) {
            return { term: this.currentTerm, success: false };
        }
        
        if (term > this.currentTerm) {
            this.currentTerm = term;
            this.votedFor = null;
        }
        
        this.state = 'follower';
        this.resetElectionTimer();
        
        // Check log consistency
        if (prevLogIndex > 0) {
            if (this.log.length < prevLogIndex || 
                this.log[prevLogIndex - 1].term !== prevLogTerm) {
                return { term: this.currentTerm, success: false };
            }
        }
        
        // Append new entries
        if (entries && entries.length > 0) {
            this.log = this.log.slice(0, prevLogIndex).concat(entries);
        }
        
        // Update commit index
        if (leaderCommit > this.commitIndex) {
            this.commitIndex = Math.min(leaderCommit, this.log.length);
            this.applyCommittedEntries();
        }
        
        return { term: this.currentTerm, success: true };
    }
    
    // Start election
    startElection() {
        this.state = 'candidate';
        this.currentTerm++;
        this.votedFor = this.id;
        this.resetElectionTimer();
        
        let votes = 1; // Vote for self
        const majority = Math.floor(this.peers.length / 2) + 1;
        
        // Request votes from peers
        const promises = this.peers.map(peer => 
            peer.requestVote(
                this.currentTerm,
                this.id,
                this.log.length,
                this.log.length > 0 ? this.log[this.log.length - 1].term : 0
            )
        );
        
        Promise.all(promises).then(responses => {
            if (this.state !== 'candidate') return;
            
            for (const response of responses) {
                if (response.term > this.currentTerm) {
                    this.currentTerm = response.term;
                    this.state = 'follower';
                    this.votedFor = null;
                    return;
                }
                
                if (response.voteGranted) {
                    votes++;
                }
            }
            
            if (votes >= majority) {
                this.becomeLeader();
            }
        });
    }
    
    // Become leader
    becomeLeader() {
        this.state = 'leader';
        
        // Initialize leader state
        for (const peer of this.peers) {
            this.nextIndex.set(peer.id, this.log.length + 1);
            this.matchIndex.set(peer.id, 0);
        }
        
        // Send heartbeats
        this.sendHeartbeats();
    }
    
    // Send heartbeats/append entries
    sendHeartbeats() {
        if (this.state !== 'leader') return;
        
        for (const peer of this.peers) {
            const prevLogIndex = this.nextIndex.get(peer.id) - 1;
            const prevLogTerm = prevLogIndex > 0 ? this.log[prevLogIndex - 1].term : 0;
            const entries = this.log.slice(prevLogIndex);
            
            peer.appendEntries(
                this.currentTerm,
                this.id,
                prevLogIndex,
                prevLogTerm,
                entries,
                this.commitIndex
            ).then(response => {
                if (response.term > this.currentTerm) {
                    this.currentTerm = response.term;
                    this.state = 'follower';
                    this.votedFor = null;
                } else if (response.success) {
                    this.nextIndex.set(peer.id, prevLogIndex + entries.length + 1);
                    this.matchIndex.set(peer.id, prevLogIndex + entries.length);
                    this.updateCommitIndex();
                } else {
                    // Decrement nextIndex and retry
                    this.nextIndex.set(peer.id, Math.max(1, this.nextIndex.get(peer.id) - 1));
                }
            });
        }
        
        // Schedule next heartbeat
        setTimeout(() => this.sendHeartbeats(), 50);
    }
    
    // Update commit index based on majority
    updateCommitIndex() {
        const matchIndexes = Array.from(this.matchIndex.values());
        matchIndexes.push(this.log.length); // Include leader
        matchIndexes.sort((a, b) => b - a);
        
        const majority = Math.floor((this.peers.length + 1) / 2);
        const newCommitIndex = matchIndexes[majority - 1];
        
        if (newCommitIndex > this.commitIndex && 
            this.log[newCommitIndex - 1].term === this.currentTerm) {
            this.commitIndex = newCommitIndex;
            this.applyCommittedEntries();
        }
    }
    
    // Apply committed entries to state machine
    applyCommittedEntries() {
        while (this.lastApplied < this.commitIndex) {
            this.lastApplied++;
            const entry = this.log[this.lastApplied - 1];
            // Apply entry to state machine
            console.log(`Applied entry ${this.lastApplied}:`, entry);
        }
    }
    
    // Check if candidate's log is up-to-date
    isLogUpToDate(lastLogIndex, lastLogTerm) {
        const ourLastTerm = this.log.length > 0 ? 
            this.log[this.log.length - 1].term : 0;
        
        if (lastLogTerm !== ourLastTerm) {
            return lastLogTerm > ourLastTerm;
        }
        
        return lastLogIndex >= this.log.length;
    }
    
    // Reset election timer
    resetElectionTimer() {
        clearTimeout(this.electionTimer);
        const timeout = 150 + Math.random() * 150; // 150-300ms
        
        this.electionTimer = setTimeout(() => {
            if (this.state !== 'leader') {
                this.startElection();
            }
        }, timeout);
    }
}
```

## Distributed Coordination

### Distributed Lock Manager
```javascript
/**
 * Distributed lock implementation using Redis-like approach
 */
class DistributedLockManager {
    constructor(nodes) {
        this.nodes = nodes;
        this.locks = new Map();
        this.lockTimeout = 30000; // 30 seconds default
    }
    
    async acquireLock(resource, clientId, timeout = this.lockTimeout) {
        const lockId = `${resource}:${Date.now()}:${clientId}`;
        const expiryTime = Date.now() + timeout;
        
        // Try to acquire lock on majority of nodes
        const promises = this.nodes.map(node => 
            this.tryLockOnNode(node, resource, lockId, expiryTime)
        );
        
        const results = await Promise.allSettled(promises);
        const successCount = results.filter(r => 
            r.status === 'fulfilled' && r.value === true
        ).length;
        
        const majority = Math.floor(this.nodes.length / 2) + 1;
        
        if (successCount >= majority) {
            this.locks.set(resource, {
                lockId,
                expiryTime,
                nodes: successCount
            });
            
            // Set up auto-release
            setTimeout(() => this.releaseLock(resource, clientId), timeout);
            
            return lockId;
        } else {
            // Failed to acquire - release partial locks
            await this.releaseLock(resource, clientId);
            return null;
        }
    }
    
    async tryLockOnNode(node, resource, lockId, expiryTime) {
        // Check if resource is already locked
        const currentLock = await node.get(resource);
        
        if (currentLock && currentLock.expiryTime > Date.now()) {
            return false; // Already locked
        }
        
        // Try to set lock atomically
        return await node.setIfNotExists(resource, {
            lockId,
            expiryTime
        });
    }
    
    async releaseLock(resource, clientId) {
        const lock = this.locks.get(resource);
        if (!lock) return false;
        
        // Release from all nodes
        const promises = this.nodes.map(node => 
            node.delete(resource, lock.lockId)
        );
        
        await Promise.allSettled(promises);
        this.locks.delete(resource);
        
        return true;
    }
    
    async extendLock(resource, clientId, additionalTime) {
        const lock = this.locks.get(resource);
        if (!lock) return false;
        
        const newExpiry = lock.expiryTime + additionalTime;
        
        // Update on majority of nodes
        const promises = this.nodes.map(node => 
            node.updateExpiry(resource, lock.lockId, newExpiry)
        );
        
        const results = await Promise.allSettled(promises);
        const successCount = results.filter(r => 
            r.status === 'fulfilled' && r.value === true
        ).length;
        
        const majority = Math.floor(this.nodes.length / 2) + 1;
        
        if (successCount >= majority) {
            lock.expiryTime = newExpiry;
            return true;
        }
        
        return false;
    }
}
```

### Vector Clocks
```javascript
/**
 * Vector clock for tracking causality in distributed systems
 */
class VectorClock {
    constructor(nodeId) {
        this.nodeId = nodeId;
        this.clock = new Map();
        this.clock.set(nodeId, 0);
    }
    
    // Increment local clock
    increment() {
        this.clock.set(this.nodeId, this.clock.get(this.nodeId) + 1);
        return this.toObject();
    }
    
    // Update clock based on received message
    update(otherClock) {
        // Update with max of each component
        for (const [nodeId, timestamp] of Object.entries(otherClock)) {
            const currentTime = this.clock.get(nodeId) || 0;
            this.clock.set(nodeId, Math.max(currentTime, timestamp));
        }
        
        // Increment local component
        this.increment();
        
        return this.toObject();
    }
    
    // Compare two vector clocks
    compare(other) {
        let isLess = false;
        let isGreater = false;
        
        const allNodes = new Set([
            ...this.clock.keys(),
            ...Object.keys(other)
        ]);
        
        for (const nodeId of allNodes) {
            const thisTime = this.clock.get(nodeId) || 0;
            const otherTime = other[nodeId] || 0;
            
            if (thisTime < otherTime) {
                isLess = true;
            } else if (thisTime > otherTime) {
                isGreater = true;
            }
        }
        
        if (isLess && !isGreater) {
            return -1; // This happened before other
        } else if (isGreater && !isLess) {
            return 1; // This happened after other
        } else if (!isLess && !isGreater) {
            return 0; // Same or equal
        } else {
            return null; // Concurrent (no causal relationship)
        }
    }
    
    toObject() {
        const obj = {};
        for (const [k, v] of this.clock) {
            obj[k] = v;
        }
        return obj;
    }
}
```

## Message Passing and RPC

### Request-Reply Pattern
```javascript
/**
 * Reliable request-reply with retries and timeout
 */
class ReliableMessaging {
    constructor(options = {}) {
        this.timeout = options.timeout || 5000;
        this.maxRetries = options.maxRetries || 3;
        this.retryDelay = options.retryDelay || 1000;
        this.messages = new Map();
    }
    
    async sendRequest(destination, message) {
        const messageId = this.generateMessageId();
        const request = {
            id: messageId,
            destination,
            payload: message,
            timestamp: Date.now(),
            retries: 0
        };
        
        this.messages.set(messageId, request);
        
        return await this.sendWithRetry(request);
    }
    
    async sendWithRetry(request) {
        while (request.retries < this.maxRetries) {
            try {
                const response = await this.send(request);
                
                if (response) {
                    this.messages.delete(request.id);
                    return response;
                }
            } catch (error) {
                request.retries++;
                
                if (request.retries >= this.maxRetries) {
                    this.messages.delete(request.id);
                    throw new Error(`Failed after ${this.maxRetries} retries: ${error.message}`);
                }
                
                // Exponential backoff
                await this.delay(this.retryDelay * Math.pow(2, request.retries - 1));
            }
        }
    }
    
    async send(request) {
        return new Promise((resolve, reject) => {
            const timer = setTimeout(() => {
                reject(new Error('Request timeout'));
            }, this.timeout);
            
            // Simulate network send
            this.networkSend(request.destination, request.payload)
                .then(response => {
                    clearTimeout(timer);
                    resolve(response);
                })
                .catch(error => {
                    clearTimeout(timer);
                    reject(error);
                });
        });
    }
    
    async networkSend(destination, payload) {
        // Simulate network latency and possible failure
        await this.delay(Math.random() * 100);
        
        if (Math.random() < 0.1) {
            throw new Error('Network error');
        }
        
        return { success: true, data: payload };
    }
    
    generateMessageId() {
        return `msg-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}
```

## AWS Distributed Patterns

### Multi-Region Active-Active
```javascript
/**
 * Multi-region active-active architecture
 */
class MultiRegionSystem {
    constructor() {
        this.regions = new Map();
        this.routingPolicy = 'latency'; // latency, geolocation, weighted
    }
    
    addRegion(name, config) {
        this.regions.set(name, {
            name,
            endpoint: config.endpoint,
            weight: config.weight || 1,
            healthy: true,
            latency: new Map(),
            database: config.database,
            cache: config.cache
        });
    }
    
    async route(request) {
        const region = this.selectRegion(request);
        
        if (!region) {
            throw new Error('No healthy regions available');
        }
        
        try {
            return await this.processInRegion(region, request);
        } catch (error) {
            // Failover to next best region
            return await this.failover(request, region);
        }
    }
    
    selectRegion(request) {
        const healthyRegions = Array.from(this.regions.values())
            .filter(r => r.healthy);
        
        if (healthyRegions.length === 0) return null;
        
        switch (this.routingPolicy) {
            case 'latency':
                return this.selectByLatency(request, healthyRegions);
            case 'geolocation':
                return this.selectByGeolocation(request, healthyRegions);
            case 'weighted':
                return this.selectByWeight(healthyRegions);
            default:
                return healthyRegions[0];
        }
    }
    
    selectByLatency(request, regions) {
        // Select region with lowest latency from client
        let bestRegion = regions[0];
        let lowestLatency = Infinity;
        
        for (const region of regions) {
            const latency = this.estimateLatency(request.clientLocation, region.name);
            if (latency < lowestLatency) {
                lowestLatency = latency;
                bestRegion = region;
            }
        }
        
        return bestRegion;
    }
    
    selectByGeolocation(request, regions) {
        // Map client location to nearest region
        const locationMap = {
            'US': 'us-east-1',
            'EU': 'eu-west-1',
            'ASIA': 'ap-southeast-1'
        };
        
        const preferredRegion = locationMap[request.clientLocation];
        return regions.find(r => r.name === preferredRegion) || regions[0];
    }
    
    selectByWeight(regions) {
        // Weighted random selection
        const totalWeight = regions.reduce((sum, r) => sum + r.weight, 0);
        let random = Math.random() * totalWeight;
        
        for (const region of regions) {
            random -= region.weight;
            if (random <= 0) {
                return region;
            }
        }
        
        return regions[0];
    }
    
    async processInRegion(region, request) {
        // Check cache first
        const cached = await region.cache.get(request.key);
        if (cached) return cached;
        
        // Process request
        const result = await this.processRequest(request);
        
        // Update cache
        await region.cache.set(request.key, result);
        
        // Replicate to other regions asynchronously
        this.replicateToRegions(request.key, result, region.name);
        
        return result;
    }
    
    async replicateToRegions(key, value, sourceRegion) {
        const otherRegions = Array.from(this.regions.values())
            .filter(r => r.name !== sourceRegion && r.healthy);
        
        const promises = otherRegions.map(region => 
            this.replicateToRegion(region, key, value)
                .catch(error => console.error(`Replication to ${region.name} failed:`, error))
        );
        
        // Fire and forget
        Promise.all(promises);
    }
    
    async replicateToRegion(region, key, value) {
        // Update cache
        await region.cache.set(key, value);
        
        // Update database with conflict resolution
        await this.resolveAndWrite(region.database, key, value);
    }
    
    async resolveAndWrite(database, key, value) {
        const existing = await database.get(key);
        
        if (!existing || this.shouldOverwrite(existing, value)) {
            await database.set(key, value);
        }
    }
    
    shouldOverwrite(existing, newValue) {
        // Last-write-wins conflict resolution
        return newValue.timestamp > existing.timestamp;
    }
    
    estimateLatency(clientLocation, regionName) {
        // Simplified latency estimation
        const latencyMap = {
            'US': { 'us-east-1': 10, 'eu-west-1': 100, 'ap-southeast-1': 150 },
            'EU': { 'us-east-1': 100, 'eu-west-1': 10, 'ap-southeast-1': 120 },
            'ASIA': { 'us-east-1': 150, 'eu-west-1': 120, 'ap-southeast-1': 10 }
        };
        
        return latencyMap[clientLocation]?.[regionName] || 200;
    }
    
    async failover(request, failedRegion) {
        console.log(`Failing over from ${failedRegion.name}`);
        failedRegion.healthy = false;
        
        // Schedule health check
        setTimeout(() => this.healthCheck(failedRegion), 30000);
        
        // Retry with different region
        return await this.route(request);
    }
    
    async healthCheck(region) {
        try {
            await this.ping(region);
            region.healthy = true;
            console.log(`Region ${region.name} is healthy again`);
        } catch (error) {
            // Still unhealthy, check again later
            setTimeout(() => this.healthCheck(region), 30000);
        }
    }
    
    async ping(region) {
        // Implement health check logic
        return true;
    }
    
    async processRequest(request) {
        // Simulate request processing
        return {
            success: true,
            data: request,
            timestamp: Date.now()
        };
    }
}
```

## Interview Tips

### Common Questions
1. **"Explain CAP theorem with examples"**
   - CP: Banking systems (consistency critical)
   - AP: Social media feeds (availability critical)
   - CA: Not possible in distributed systems

2. **"How does consensus work?"**
   - Leader election
   - Log replication
   - Majority agreement
   - Split-brain prevention

3. **"Handling network partitions?"**
   - Quorum-based decisions
   - Eventual consistency
   - Conflict resolution strategies

### Red Flags to Avoid
- ❌ Assuming network is reliable
- ❌ Ignoring partial failures
- ❌ Not planning for split-brain
- ❌ Over-simplifying consistency models

### AWS-Specific Follow-ups
- "Design global DynamoDB tables"
- "Implement cross-region replication"
- "Handle Route 53 failover"
- "Design event-driven architecture with EventBridge"

## Key Takeaways
1. **CAP theorem** forces trade-offs in distributed systems
2. **Consensus algorithms** ensure agreement despite failures
3. **Vector clocks** track causality in distributed events
4. **Partition tolerance** is non-negotiable in cloud systems
5. **AWS services** abstract many distributed system complexities

---

*Previous: [← Caching and Performance](./09-caching-and-performance.md)*
*Back to: [README →](./README.md)*
