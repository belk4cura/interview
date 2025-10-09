# Caching and Performance

## Overview
Caching is a critical optimization technique that stores frequently accessed data in fast storage layers. Understanding caching strategies and performance optimization is essential for building scalable AWS applications.

### Key Concepts
- **Cache Levels**: L1/L2/L3, Memory, Distributed
- **Cache Strategies**: LRU, LFU, TTL, Write-through, Write-back
- **Performance Metrics**: Hit ratio, Latency, Throughput
- **AWS Applications**: ElastiCache, CloudFront, DynamoDB DAX

## Cache Implementation Strategies

### LRU Cache (Least Recently Used)
```javascript
/**
 * LRU Cache with O(1) operations
 * Using Map to maintain insertion order
 */
class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.cache = new Map();
    }
    
    get(key) {
        if (!this.cache.has(key)) {
            return -1;
        }
        
        // Move to end (most recent)
        const value = this.cache.get(key);
        this.cache.delete(key);
        this.cache.set(key, value);
        
        return value;
    }
    
    put(key, value) {
        // Remove if exists (to update position)
        if (this.cache.has(key)) {
            this.cache.delete(key);
        }
        
        // Check capacity
        if (this.cache.size >= this.capacity) {
            // Remove least recently used (first item)
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        
        this.cache.set(key, value);
    }
    
    // Get cache statistics
    getStats() {
        return {
            size: this.cache.size,
            capacity: this.capacity,
            utilization: (this.cache.size / this.capacity * 100).toFixed(2) + '%'
        };
    }
}

// Usage
const cache = new LRUCache(3);
cache.put('user:1', { name: 'Alice' });
cache.put('user:2', { name: 'Bob' });
cache.put('user:3', { name: 'Charlie' });
cache.get('user:1'); // Moves user:1 to most recent
cache.put('user:4', { name: 'David' }); // Evicts user:2
```

### LFU Cache (Least Frequently Used)
```javascript
/**
 * LFU Cache - evicts least frequently used items
 * All operations O(1)
 */
class LFUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.minFreq = 0;
        this.keyToVal = new Map();
        this.keyToFreq = new Map();
        this.freqToKeys = new Map();
    }
    
    get(key) {
        if (!this.keyToVal.has(key)) return -1;
        
        this.increaseFreq(key);
        return this.keyToVal.get(key);
    }
    
    put(key, value) {
        if (this.capacity === 0) return;
        
        if (this.keyToVal.has(key)) {
            this.keyToVal.set(key, value);
            this.increaseFreq(key);
            return;
        }
        
        if (this.keyToVal.size >= this.capacity) {
            this.evict();
        }
        
        this.keyToVal.set(key, value);
        this.keyToFreq.set(key, 1);
        
        if (!this.freqToKeys.has(1)) {
            this.freqToKeys.set(1, new Set());
        }
        this.freqToKeys.get(1).add(key);
        
        this.minFreq = 1;
    }
    
    increaseFreq(key) {
        const freq = this.keyToFreq.get(key);
        this.keyToFreq.set(key, freq + 1);
        
        // Remove from old frequency list
        this.freqToKeys.get(freq).delete(key);
        if (this.freqToKeys.get(freq).size === 0) {
            this.freqToKeys.delete(freq);
            if (this.minFreq === freq) {
                this.minFreq++;
            }
        }
        
        // Add to new frequency list
        if (!this.freqToKeys.has(freq + 1)) {
            this.freqToKeys.set(freq + 1, new Set());
        }
        this.freqToKeys.get(freq + 1).add(key);
    }
    
    evict() {
        const keySet = this.freqToKeys.get(this.minFreq);
        const keyToEvict = keySet.values().next().value;
        
        keySet.delete(keyToEvict);
        if (keySet.size === 0) {
            this.freqToKeys.delete(this.minFreq);
        }
        
        this.keyToVal.delete(keyToEvict);
        this.keyToFreq.delete(keyToEvict);
    }
}
```

### TTL Cache (Time To Live)
```javascript
/**
 * Cache with automatic expiration
 */
class TTLCache {
    constructor(defaultTTL = 3600000) { // 1 hour default
        this.cache = new Map();
        this.timers = new Map();
        this.defaultTTL = defaultTTL;
        this.stats = {
            hits: 0,
            misses: 0,
            evictions: 0
        };
    }
    
    set(key, value, ttl = this.defaultTTL) {
        // Clear existing timer if any
        if (this.timers.has(key)) {
            clearTimeout(this.timers.get(key));
        }
        
        this.cache.set(key, {
            value,
            expiresAt: Date.now() + ttl
        });
        
        // Set expiration timer
        const timer = setTimeout(() => {
            this.delete(key);
            this.stats.evictions++;
        }, ttl);
        
        this.timers.set(key, timer);
    }
    
    get(key) {
        const item = this.cache.get(key);
        
        if (!item) {
            this.stats.misses++;
            return undefined;
        }
        
        if (Date.now() > item.expiresAt) {
            this.delete(key);
            this.stats.misses++;
            this.stats.evictions++;
            return undefined;
        }
        
        this.stats.hits++;
        return item.value;
    }
    
    delete(key) {
        if (this.timers.has(key)) {
            clearTimeout(this.timers.get(key));
            this.timers.delete(key);
        }
        return this.cache.delete(key);
    }
    
    getHitRate() {
        const total = this.stats.hits + this.stats.misses;
        return total === 0 ? 0 : (this.stats.hits / total * 100).toFixed(2);
    }
    
    clear() {
        for (const timer of this.timers.values()) {
            clearTimeout(timer);
        }
        this.cache.clear();
        this.timers.clear();
    }
}
```

## Write Strategies

### Write-Through Cache
```javascript
/**
 * Writes go through cache to database
 */
class WriteThroughCache {
    constructor(cache, database) {
        this.cache = cache;
        this.database = database;
    }
    
    async write(key, value) {
        // Write to database first
        await this.database.write(key, value);
        
        // Then update cache
        this.cache.set(key, value);
        
        return true;
    }
    
    async read(key) {
        // Try cache first
        let value = this.cache.get(key);
        
        if (value === undefined) {
            // Cache miss - read from database
            value = await this.database.read(key);
            
            if (value !== undefined) {
                // Update cache
                this.cache.set(key, value);
            }
        }
        
        return value;
    }
}
```

### Write-Back Cache (Write-Behind)
```javascript
/**
 * Writes to cache immediately, database eventually
 */
class WriteBackCache {
    constructor(cache, database, flushInterval = 5000) {
        this.cache = cache;
        this.database = database;
        this.dirtyKeys = new Set();
        this.flushInterval = flushInterval;
        
        // Periodic flush
        this.startFlushTimer();
    }
    
    write(key, value) {
        // Write to cache immediately
        this.cache.set(key, value);
        this.dirtyKeys.add(key);
        
        return true;
    }
    
    async read(key) {
        // Always read from cache
        let value = this.cache.get(key);
        
        if (value === undefined) {
            // Cache miss - read from database
            value = await this.database.read(key);
            
            if (value !== undefined) {
                this.cache.set(key, value);
            }
        }
        
        return value;
    }
    
    async flush() {
        const keysToFlush = Array.from(this.dirtyKeys);
        this.dirtyKeys.clear();
        
        const promises = keysToFlush.map(key => {
            const value = this.cache.get(key);
            return this.database.write(key, value);
        });
        
        await Promise.all(promises);
        
        return keysToFlush.length;
    }
    
    startFlushTimer() {
        setInterval(() => {
            if (this.dirtyKeys.size > 0) {
                this.flush();
            }
        }, this.flushInterval);
    }
}
```

## Distributed Caching

### Consistent Hash Ring
```javascript
/**
 * Distributed cache with consistent hashing
 */
class DistributedCache {
    constructor(nodes) {
        this.nodes = new Map();
        this.ring = new Map();
        this.virtualNodes = 150;
        
        for (const node of nodes) {
            this.addNode(node);
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
    
    addNode(node) {
        this.nodes.set(node.id, node);
        
        // Add virtual nodes
        for (let i = 0; i < this.virtualNodes; i++) {
            const virtualKey = `${node.id}:${i}`;
            const hash = this.hash(virtualKey);
            this.ring.set(hash, node);
        }
    }
    
    removeNode(nodeId) {
        if (!this.nodes.has(nodeId)) return;
        
        // Remove virtual nodes
        for (let i = 0; i < this.virtualNodes; i++) {
            const virtualKey = `${nodeId}:${i}`;
            const hash = this.hash(virtualKey);
            this.ring.delete(hash);
        }
        
        this.nodes.delete(nodeId);
    }
    
    getNode(key) {
        if (this.ring.size === 0) return null;
        
        const keyHash = this.hash(key);
        const sortedHashes = Array.from(this.ring.keys()).sort((a, b) => a - b);
        
        // Find first hash >= key hash
        let selectedHash = sortedHashes[0];
        for (const hash of sortedHashes) {
            if (hash >= keyHash) {
                selectedHash = hash;
                break;
            }
        }
        
        return this.ring.get(selectedHash);
    }
    
    async get(key) {
        const node = this.getNode(key);
        if (!node) return null;
        
        return await node.get(key);
    }
    
    async set(key, value, ttl) {
        const node = this.getNode(key);
        if (!node) return false;
        
        return await node.set(key, value, ttl);
    }
}
```

### Cache Replication
```javascript
/**
 * Replicated cache for high availability
 */
class ReplicatedCache {
    constructor(nodes, replicationFactor = 3) {
        this.nodes = nodes;
        this.replicationFactor = Math.min(replicationFactor, nodes.length);
    }
    
    getReplicaNodes(key) {
        const hash = this.hash(key);
        const startIndex = hash % this.nodes.length;
        const replicas = [];
        
        for (let i = 0; i < this.replicationFactor; i++) {
            const index = (startIndex + i) % this.nodes.length;
            replicas.push(this.nodes[index]);
        }
        
        return replicas;
    }
    
    async set(key, value, options = {}) {
        const replicas = this.getReplicaNodes(key);
        const promises = replicas.map(node => node.set(key, value));
        
        if (options.consistency === 'strong') {
            // Wait for all replicas
            await Promise.all(promises);
        } else if (options.consistency === 'eventual') {
            // Wait for primary only
            await promises[0];
        } else {
            // Quorum - wait for majority
            const quorum = Math.floor(this.replicationFactor / 2) + 1;
            await Promise.race([
                this.waitForQuorum(promises, quorum),
                this.timeout(options.timeout || 5000)
            ]);
        }
        
        return true;
    }
    
    async get(key) {
        const replicas = this.getReplicaNodes(key);
        
        // Try replicas in order
        for (const node of replicas) {
            try {
                const value = await node.get(key);
                if (value !== undefined) {
                    return value;
                }
            } catch (error) {
                // Try next replica
                continue;
            }
        }
        
        return undefined;
    }
    
    async waitForQuorum(promises, quorum) {
        let successCount = 0;
        const results = await Promise.allSettled(promises);
        
        for (const result of results) {
            if (result.status === 'fulfilled') {
                successCount++;
                if (successCount >= quorum) {
                    return true;
                }
            }
        }
        
        throw new Error('Failed to achieve quorum');
    }
    
    timeout(ms) {
        return new Promise((_, reject) => 
            setTimeout(() => reject(new Error('Timeout')), ms)
        );
    }
    
    hash(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
        }
        return Math.abs(hash);
    }
}
```

## Performance Optimization

### Request Coalescing
```javascript
/**
 * Combine multiple requests for same key
 */
class RequestCoalescer {
    constructor(fetcher) {
        this.fetcher = fetcher;
        this.inFlight = new Map();
    }
    
    async get(key) {
        // Check if request is already in flight
        if (this.inFlight.has(key)) {
            return await this.inFlight.get(key);
        }
        
        // Create new request
        const promise = this.fetcher(key).finally(() => {
            this.inFlight.delete(key);
        });
        
        this.inFlight.set(key, promise);
        
        return await promise;
    }
}
```

### Batch Processing
```javascript
/**
 * Batch multiple operations for efficiency
 */
class BatchProcessor {
    constructor(processor, options = {}) {
        this.processor = processor;
        this.batchSize = options.batchSize || 100;
        this.batchTimeout = options.batchTimeout || 100;
        this.queue = [];
        this.promises = [];
        this.timer = null;
    }
    
    async process(item) {
        return new Promise((resolve, reject) => {
            this.queue.push({ item, resolve, reject });
            this.promises.push({ resolve, reject });
            
            if (this.queue.length >= this.batchSize) {
                this.flush();
            } else if (!this.timer) {
                this.timer = setTimeout(() => this.flush(), this.batchTimeout);
            }
        });
    }
    
    async flush() {
        if (this.timer) {
            clearTimeout(this.timer);
            this.timer = null;
        }
        
        if (this.queue.length === 0) return;
        
        const batch = this.queue.splice(0);
        const promises = this.promises.splice(0);
        
        try {
            const items = batch.map(b => b.item);
            const results = await this.processor(items);
            
            for (let i = 0; i < batch.length; i++) {
                batch[i].resolve(results[i]);
            }
        } catch (error) {
            for (const { reject } of batch) {
                reject(error);
            }
        }
    }
}
```

## AWS Cache Services

### ElastiCache Patterns
```javascript
/**
 * ElastiCache implementation patterns
 */
class ElastiCacheManager {
    constructor(config) {
        this.endpoint = config.endpoint;
        this.port = config.port;
        this.cluster = config.cluster;
    }
    
    // Lazy loading pattern
    async lazyLoad(key, loader) {
        let value = await this.get(key);
        
        if (value === null) {
            // Cache miss - load from source
            value = await loader();
            
            if (value !== null) {
                await this.set(key, value, 3600); // 1 hour TTL
            }
        }
        
        return value;
    }
    
    // Write-through pattern
    async writeThrough(key, value, writer) {
        // Write to database
        await writer(value);
        
        // Update cache
        await this.set(key, value);
        
        return true;
    }
    
    // Cache warming
    async warmCache(keys, loader) {
        const promises = keys.map(async key => {
            const value = await loader(key);
            if (value !== null) {
                await this.set(key, value);
            }
        });
        
        await Promise.all(promises);
    }
    
    // Circuit breaker for cache
    async getWithFallback(key, fallback) {
        try {
            const value = await this.get(key);
            if (value !== null) return value;
        } catch (error) {
            console.error('Cache error:', error);
        }
        
        // Fallback to source
        return await fallback();
    }
}
```

### CloudFront Edge Caching
```javascript
/**
 * CloudFront cache optimization
 */
class CloudFrontOptimizer {
    constructor() {
        this.behaviors = [];
    }
    
    addCacheBehavior(pattern, config) {
        this.behaviors.push({
            pathPattern: pattern,
            targetOriginId: config.origin,
            viewerProtocolPolicy: config.protocol || 'redirect-to-https',
            cachePolicyId: this.getCachePolicy(config),
            compress: config.compress !== false
        });
    }
    
    getCachePolicy(config) {
        // Determine cache policy based on content type
        const policies = {
            static: {
                defaultTTL: 86400,    // 1 day
                maxTTL: 31536000,     // 1 year
                minTTL: 1
            },
            dynamic: {
                defaultTTL: 0,
                maxTTL: 0,
                minTTL: 0
            },
            api: {
                defaultTTL: 60,       // 1 minute
                maxTTL: 300,          // 5 minutes
                minTTL: 0
            }
        };
        
        return policies[config.type] || policies.static;
    }
    
    generateCacheKey(request) {
        const components = [];
        
        // URI
        components.push(request.uri);
        
        // Query strings (sorted for consistency)
        if (request.queryString) {
            const sorted = Object.keys(request.queryString)
                .sort()
                .map(k => `${k}=${request.queryString[k]}`)
                .join('&');
            components.push(sorted);
        }
        
        // Headers (if configured)
        if (request.headers) {
            components.push(JSON.stringify(request.headers));
        }
        
        return components.join('::');
    }
}
```

## Interview Tips

### Common Questions
1. **"Cache eviction policies?"**
   - LRU: Remove least recently used
   - LFU: Remove least frequently used
   - FIFO: Remove oldest
   - Random: Remove random item

2. **"Cache consistency strategies?"**
   - Strong consistency: All reads see latest write
   - Eventual consistency: Reads eventually see latest
   - Weak consistency: No guarantees

3. **"Cache aside vs cache through?"**
   - Cache aside: App manages cache
   - Cache through: Cache manages database
   - Trade-offs in complexity vs consistency

### Red Flags to Avoid
- ❌ Not handling cache stampede
- ❌ Ignoring cache invalidation complexity
- ❌ Over-caching dynamic content
- ❌ Not monitoring hit rates

### AWS-Specific Follow-ups
- "Design multi-tier caching strategy"
- "Optimize CloudFront for global distribution"
- "Implement cache warming for ElastiCache"
- "Handle cache failures gracefully"

## Key Takeaways
1. **Cache strategically** - Not everything benefits from caching
2. **Monitor hit rates** - Low hit rate means wasted resources
3. **Plan invalidation** - Stale data is worse than slow data
4. **Layer caches** - Browser, CDN, application, database
5. **Handle failures** - Always have fallback to source

---

*Previous: [← Scalability Patterns](./08-scalability-patterns.md)*
*Next: [Distributed Systems →](./10-distributed-systems.md)*
