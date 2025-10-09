# Hash Maps and Sets

## Plain English Explanation
A **hash map** is like a library's card catalog system. Instead of searching through every book (O(n)), you go directly to the right drawer using the book's title. The librarian (hash function) tells you exactly which drawer number to check. Sometimes two books might be assigned to the same drawer (collision), so you need a system to handle that - like having folders within the drawer.

A **set** is like a guest list at an exclusive party - you only care if someone's name is on the list or not. Each person can only be listed once (unique values), and you can quickly check if they're invited.

## Visual Representation
```
Hash Table (Array + Hash Function):

Key: "name"  →  hash("name") = 2  →  [0][1][2][3][4]
                                           ↓
                                       ["name", "John"]

Collision Handling (Chaining):
[0] → null
[1] → [["age", 25]]
[2] → [["name", "John"]] → [["city", "NYC"]]  (collision!)
[3] → [["id", 123]]
[4] → null

Set Operations:
Set A: {1, 2, 3}     Set B: {2, 3, 4}
Union:        A ∪ B = {1, 2, 3, 4}
Intersection: A ∩ B = {2, 3}
Difference:   A - B = {1}
```

## Overview
Hash tables are one of the most important data structures in computer science, providing O(1) average-case operations for insert, delete, and search. They form the foundation of many system designs and are extensively used in caching and indexing.

### Key Concepts
- **Hash Functions**: Maps keys to array indices
- **Collision Resolution**: Handling when multiple keys hash to same index
- **Load Factor**: Ratio of filled slots affecting performance
- **AWS Applications**: ElastiCache, DynamoDB hash keys, CloudFront caching

### When to Use What?
- **Use Hash Map when**: Need key-value pairs, fast lookups, caching, counting occurrences
- **Use Set when**: Need unique values only, membership testing, removing duplicates
- **Avoid Hash Map when**: Need ordered iteration, finding min/max frequently
- **Avoid Set when**: Need to count occurrences, maintain insertion order

## Hash Table Implementation

### Basic Hash Table with Chaining

```javascript
class HashTable {
    constructor(size = 53) {
        this.keyMap = new Array(size);  // Prime size reduces collisions
        this.size = 0;  // Track number of key-value pairs
    }
    
    // ============= HASH FUNCTION =============
    // Time: O(k) where k is key length - WHY?
    // We iterate through each character (up to 100)
    // Each iteration does O(1) operations
    // Space: O(1) - only using a few variables
    _hash(key) {
        let total = 0;
        const PRIME = 31; // WHY PRIME?
        // Primes distribute hash values more uniformly
        // Reduces clustering and collision patterns
        
        // Cap at 100 chars to ensure O(1) for very long strings
        for (let i = 0; i < Math.min(key.length, 100); i++) {
            const char = key[i];
            // Convert char to number (a=1, b=2, etc.)
            const value = char.charCodeAt(0) - 96;
            // Multiply by prime and add value
            // Mod by array length to get valid index
            total = (total * PRIME + value) % this.keyMap.length;
        }
        
        return total;
    }
    
    // ============= SET OPERATION =============
    // Time: O(1) average - WHY?
    // 1. Hash function: O(1) with char limit
    // 2. Array access: O(1) direct indexing
    // 3. Chain search: O(α) where α is load factor
    // With good hash & low load factor, α ≈ 1
    // Worst case: O(n) if all keys hash to same bucket
    set(key, value) {
        // Get bucket index in O(1)
        const index = this._hash(key);
        
        // Initialize bucket if empty
        if (!this.keyMap[index]) {
            this.keyMap[index] = [];  // Create chain for collisions
        }
        
        // Search chain for existing key - O(chain length)
        // Average chain length = load factor when distributed well
        const existing = this.keyMap[index].find(([k, v]) => k === key);
        if (existing) {
            existing[1] = value;  // Update existing value
            return;  // Don't increase size
        }
        
        // Add new pair to chain
        this.keyMap[index].push([key, value]);
        this.size++;
        
        // WHY CHECK LOAD FACTOR?
        // Load factor > 0.75 means chains getting long
        // Longer chains = slower operations
        // Resize maintains O(1) average performance
        if (this.size > this.keyMap.length * 0.75) {
            this.resize();  // O(n) but amortized O(1)
        }
    }
    
    // ============= GET OPERATION =============
    // Time: O(1) average - same reasoning as set()
    // Hash to find bucket: O(1)
    // Search bucket chain: O(load factor)
    get(key) {
        const index = this._hash(key);
        
        // Check if bucket exists
        if (this.keyMap[index]) {
            // Linear search through chain
            // Average length = load factor (< 0.75)
            const pair = this.keyMap[index].find(([k, v]) => k === key);
            return pair ? pair[1] : undefined;
        }
        
        return undefined;
    }
    
    // ============= DELETE OPERATION =============
    // Time: O(1) average for same reasons
    // Additionally uses splice which is O(chain length)
    delete(key) {
        const index = this._hash(key);
        
        if (this.keyMap[index]) {
            // Find index in chain - O(chain length)
            const pairIndex = this.keyMap[index].findIndex(([k, v]) => k === key);
            if (pairIndex !== -1) {
                // Splice is O(n) for array, but n = chain length ≈ load factor
                this.keyMap[index].splice(pairIndex, 1);
                this.size--;
                return true;
            }
        }
        
        return false;
    }
    
    // ============= RESIZE OPERATION =============
    // Time: O(n) where n is total elements - WHY?
    // Must rehash and reinsert every element
    // Space: O(n) for new array
    // This is AMORTIZED O(1) per operation - WHY?
    // Resize happens every n/2, n/4, n/8... insertions
    // Total work: n + n/2 + n/4 + ... = 2n
    // Amortized: 2n/n = O(1) per insertion
    resize() {
        const oldKeyMap = this.keyMap;
        this.keyMap = new Array(oldKeyMap.length * 2);  // Double size
        this.size = 0;  // Reset size, will recount
        
        // Rehash all elements - NECESSARY!
        // WHY? Hash function uses array length
        // New length = different hash values
        for (const bucket of oldKeyMap) {
            if (bucket) {
                for (const [key, value] of bucket) {
                    this.set(key, value);  // Rehash and insert
                }
            }
        }
    }
    
    // O(n) - returns all keys
    keys() {
        const keys = [];
        for (const bucket of this.keyMap) {
            if (bucket) {
                for (const [key, value] of bucket) {
                    keys.push(key);
                }
            }
        }
        return keys;
    }
    
    // O(n) - returns all values
    values() {
        const values = [];
        const seen = new Set();
        
        for (const bucket of this.keyMap) {
            if (bucket) {
                for (const [key, value] of bucket) {
                    if (!seen.has(value)) {
                        values.push(value);
                        seen.add(value);
                    }
                }
            }
        }
        return values;
    }
}

// Usage
const ht = new HashTable();
ht.set("firstName", "John");
ht.set("lastName", "Doe");
console.log(ht.get("firstName")); // "John"
```

### Open Addressing (Linear Probing)

```javascript
class HashTableOpenAddressing {
    constructor(size = 53) {
        this.keys = new Array(size).fill(null);
        this.values = new Array(size).fill(null);
        this.size = 0;
        this.capacity = size;
    }
    
    _hash(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = (hash + key.charCodeAt(i) * i) % this.capacity;
        }
        return hash;
    }
    
    // O(1) average - Linear probing
    set(key, value) {
        let index = this._hash(key);
        
        // Linear probing to find empty slot
        while (this.keys[index] !== null && this.keys[index] !== key) {
            index = (index + 1) % this.capacity;
        }
        
        // New key
        if (this.keys[index] === null) {
            this.size++;
        }
        
        this.keys[index] = key;
        this.values[index] = value;
        
        // Resize if load factor > 0.5
        if (this.size > this.capacity * 0.5) {
            this.resize();
        }
    }
    
    // O(1) average
    get(key) {
        let index = this._hash(key);
        
        while (this.keys[index] !== null) {
            if (this.keys[index] === key) {
                return this.values[index];
            }
            index = (index + 1) % this.capacity;
        }
        
        return undefined;
    }
    
    // O(1) average
    delete(key) {
        let index = this._hash(key);
        
        while (this.keys[index] !== null) {
            if (this.keys[index] === key) {
                this.keys[index] = null;
                this.values[index] = null;
                this.size--;
                
                // Rehash subsequent keys
                this.rehashCluster(index);
                return true;
            }
            index = (index + 1) % this.capacity;
        }
        
        return false;
    }
    
    // Rehash keys in cluster after deletion
    rehashCluster(deletedIndex) {
        let index = (deletedIndex + 1) % this.capacity;
        
        while (this.keys[index] !== null) {
            const keyToRehash = this.keys[index];
            const valueToRehash = this.values[index];
            
            this.keys[index] = null;
            this.values[index] = null;
            this.size--;
            
            this.set(keyToRehash, valueToRehash);
            index = (index + 1) % this.capacity;
        }
    }
    
    resize() {
        const oldKeys = this.keys;
        const oldValues = this.values;
        
        this.capacity = this.capacity * 2;
        this.keys = new Array(this.capacity).fill(null);
        this.values = new Array(this.capacity).fill(null);
        this.size = 0;
        
        for (let i = 0; i < oldKeys.length; i++) {
            if (oldKeys[i] !== null) {
                this.set(oldKeys[i], oldValues[i]);
            }
        }
    }
}
```

## Sets Implementation

### Hash Set

```javascript
class HashSet {
    constructor() {
        this.map = new Map();
    }
    
    // O(1) average
    add(value) {
        this.map.set(value, true);
        return this;
    }
    
    // O(1) average
    has(value) {
        return this.map.has(value);
    }
    
    // O(1) average
    delete(value) {
        return this.map.delete(value);
    }
    
    // O(n)
    clear() {
        this.map.clear();
    }
    
    // O(1)
    get size() {
        return this.map.size;
    }
    
    // Set operations
    // O(n) where n is size of this set
    union(otherSet) {
        const unionSet = new HashSet();
        
        for (const value of this.values()) {
            unionSet.add(value);
        }
        
        for (const value of otherSet.values()) {
            unionSet.add(value);
        }
        
        return unionSet;
    }
    
    // O(n) where n is size of smaller set
    intersection(otherSet) {
        const intersectionSet = new HashSet();
        const smaller = this.size <= otherSet.size ? this : otherSet;
        const larger = this.size > otherSet.size ? this : otherSet;
        
        for (const value of smaller.values()) {
            if (larger.has(value)) {
                intersectionSet.add(value);
            }
        }
        
        return intersectionSet;
    }
    
    // O(n) where n is size of this set
    difference(otherSet) {
        const differenceSet = new HashSet();
        
        for (const value of this.values()) {
            if (!otherSet.has(value)) {
                differenceSet.add(value);
            }
        }
        
        return differenceSet;
    }
    
    // O(n) where n is size of this set
    isSubsetOf(otherSet) {
        if (this.size > otherSet.size) return false;
        
        for (const value of this.values()) {
            if (!otherSet.has(value)) {
                return false;
            }
        }
        
        return true;
    }
    
    *values() {
        yield* this.map.keys();
    }
}

// Usage
const set1 = new HashSet();
set1.add(1).add(2).add(3);

const set2 = new HashSet();
set2.add(2).add(3).add(4);

console.log(set1.intersection(set2)); // {2, 3}
console.log(set1.union(set2)); // {1, 2, 3, 4}
console.log(set1.difference(set2)); // {1}
```

## Advanced Hash Concepts

### Consistent Hashing (For Distributed Systems)

```javascript
class ConsistentHash {
    constructor(nodes = [], virtualNodes = 150) {
        this.nodes = new Map();
        this.ring = new Map();
        this.virtualNodes = virtualNodes;
        
        for (const node of nodes) {
            this.addNode(node);
        }
    }
    
    // Hash function using simple string hash
    hash(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
            hash = hash & hash; // Convert to 32-bit integer
        }
        return Math.abs(hash);
    }
    
    // Add node to the ring with virtual nodes
    addNode(node) {
        this.nodes.set(node, []);
        
        for (let i = 0; i < this.virtualNodes; i++) {
            const virtualKey = `${node}:${i}`;
            const hash = this.hash(virtualKey);
            this.ring.set(hash, node);
            this.nodes.get(node).push(hash);
        }
        
        this.sortedKeys = null; // Invalidate cache
    }
    
    // Remove node from the ring
    removeNode(node) {
        if (!this.nodes.has(node)) return;
        
        const hashes = this.nodes.get(node);
        for (const hash of hashes) {
            this.ring.delete(hash);
        }
        
        this.nodes.delete(node);
        this.sortedKeys = null; // Invalidate cache
    }
    
    // Get sorted keys (cached)
    getSortedKeys() {
        if (!this.sortedKeys) {
            this.sortedKeys = Array.from(this.ring.keys()).sort((a, b) => a - b);
        }
        return this.sortedKeys;
    }
    
    // Find node responsible for a key
    getNode(key) {
        if (this.ring.size === 0) return null;
        
        const hash = this.hash(key);
        const sortedKeys = this.getSortedKeys();
        
        // Binary search for the first hash >= key hash
        let left = 0;
        let right = sortedKeys.length - 1;
        
        while (left < right) {
            const mid = Math.floor((left + right) / 2);
            if (sortedKeys[mid] < hash) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        
        // If all hashes are less than key hash, wrap around
        const selectedHash = sortedKeys[left] >= hash 
            ? sortedKeys[left] 
            : sortedKeys[0];
            
        return this.ring.get(selectedHash);
    }
    
    // Get N nodes for replication
    getNodes(key, count = 3) {
        if (this.nodes.size === 0) return [];
        
        const result = new Set();
        const hash = this.hash(key);
        const sortedKeys = this.getSortedKeys();
        
        let startIndex = 0;
        for (let i = 0; i < sortedKeys.length; i++) {
            if (sortedKeys[i] >= hash) {
                startIndex = i;
                break;
            }
        }
        
        let i = startIndex;
        while (result.size < count && result.size < this.nodes.size) {
            const node = this.ring.get(sortedKeys[i]);
            result.add(node);
            i = (i + 1) % sortedKeys.length;
        }
        
        return Array.from(result);
    }
}

// Usage - Distributed caching
const ch = new ConsistentHash(['cache1', 'cache2', 'cache3']);
console.log(ch.getNode('user:123')); // Determines which cache server
console.log(ch.getNodes('user:123', 2)); // Get primary and replica
```

### Bloom Filter (Probabilistic Set)

```javascript
class BloomFilter {
    constructor(size = 1000, hashFunctions = 3) {
        this.size = size;
        this.bitArray = new Array(size).fill(0);
        this.hashFunctions = hashFunctions;
    }
    
    // Multiple hash functions
    hash(item, seed) {
        let hash = 0;
        for (let i = 0; i < item.length; i++) {
            hash = ((hash << 5) - hash) + item.charCodeAt(i);
            hash = (hash * seed) & hash;
        }
        return Math.abs(hash) % this.size;
    }
    
    // O(k) where k is number of hash functions
    add(item) {
        for (let i = 0; i < this.hashFunctions; i++) {
            const index = this.hash(item, i + 1);
            this.bitArray[index] = 1;
        }
    }
    
    // O(k) - May have false positives, never false negatives
    mightContain(item) {
        for (let i = 0; i < this.hashFunctions; i++) {
            const index = this.hash(item, i + 1);
            if (this.bitArray[index] === 0) {
                return false; // Definitely not in set
            }
        }
        return true; // Might be in set (or false positive)
    }
    
    // Calculate false positive probability
    falsePositiveProbability() {
        const k = this.hashFunctions;
        const m = this.size;
        const n = this.bitArray.filter(bit => bit === 1).length;
        
        // (1 - e^(-k*n/m))^k
        return Math.pow(1 - Math.exp(-k * n / m), k);
    }
}

// Usage - Check if URL was visited
const bloom = new BloomFilter(10000, 3);
bloom.add("https://example.com");
bloom.add("https://aws.amazon.com");

console.log(bloom.mightContain("https://example.com")); // true
console.log(bloom.mightContain("https://google.com")); // false (or maybe true - false positive)
```

## AWS Applications

### 1. ElastiCache Implementation Pattern

```javascript
// Redis-like cache implementation
class ElastiCacheSimulator {
    constructor(maxMemory = 1000, evictionPolicy = 'LRU') {
        this.cache = new Map();
        this.maxMemory = maxMemory;
        this.currentMemory = 0;
        this.evictionPolicy = evictionPolicy;
        this.accessOrder = []; // For LRU
        this.accessCount = new Map(); // For LFU
        this.ttl = new Map(); // Time to live
    }
    
    // O(1) average - Set with TTL
    set(key, value, ttlSeconds = null) {
        const size = this.getSize(value);
        
        // Check if we need to evict
        while (this.currentMemory + size > this.maxMemory && this.cache.size > 0) {
            this.evict();
        }
        
        // Remove old value if exists
        if (this.cache.has(key)) {
            this.currentMemory -= this.getSize(this.cache.get(key));
        }
        
        this.cache.set(key, value);
        this.currentMemory += size;
        
        // Set TTL if provided
        if (ttlSeconds) {
            const expiryTime = Date.now() + (ttlSeconds * 1000);
            this.ttl.set(key, expiryTime);
        }
        
        // Update access tracking
        this.updateAccess(key);
    }
    
    // O(1) average
    get(key) {
        // Check TTL
        if (this.ttl.has(key)) {
            if (Date.now() > this.ttl.get(key)) {
                this.delete(key);
                return null;
            }
        }
        
        if (!this.cache.has(key)) {
            return null;
        }
        
        this.updateAccess(key);
        return this.cache.get(key);
    }
    
    // O(1)
    delete(key) {
        if (this.cache.has(key)) {
            this.currentMemory -= this.getSize(this.cache.get(key));
            this.cache.delete(key);
            this.ttl.delete(key);
            this.accessCount.delete(key);
            
            const index = this.accessOrder.indexOf(key);
            if (index > -1) {
                this.accessOrder.splice(index, 1);
            }
            
            return true;
        }
        return false;
    }
    
    // Eviction based on policy
    evict() {
        let keyToEvict;
        
        switch (this.evictionPolicy) {
            case 'LRU':
                keyToEvict = this.accessOrder[0];
                break;
                
            case 'LFU':
                let minCount = Infinity;
                for (const [key, count] of this.accessCount) {
                    if (count < minCount) {
                        minCount = count;
                        keyToEvict = key;
                    }
                }
                break;
                
            case 'RANDOM':
                const keys = Array.from(this.cache.keys());
                keyToEvict = keys[Math.floor(Math.random() * keys.length)];
                break;
                
            default:
                keyToEvict = this.cache.keys().next().value; // FIFO
        }
        
        if (keyToEvict) {
            this.delete(keyToEvict);
        }
    }
    
    updateAccess(key) {
        // LRU tracking
        const index = this.accessOrder.indexOf(key);
        if (index > -1) {
            this.accessOrder.splice(index, 1);
        }
        this.accessOrder.push(key);
        
        // LFU tracking
        this.accessCount.set(key, (this.accessCount.get(key) || 0) + 1);
    }
    
    getSize(value) {
        // Simplified size calculation
        return JSON.stringify(value).length;
    }
    
    // Cache statistics
    getStats() {
        return {
            keys: this.cache.size,
            memory: this.currentMemory,
            maxMemory: this.maxMemory,
            utilization: (this.currentMemory / this.maxMemory * 100).toFixed(2) + '%'
        };
    }
}

// Usage
const cache = new ElastiCacheSimulator(1000, 'LRU');
cache.set('user:123', { name: 'John', age: 30 }, 300); // 5 min TTL
cache.set('session:abc', { token: 'xyz' }, 3600); // 1 hour TTL
console.log(cache.get('user:123')); // Updates LRU order
```

### 2. DynamoDB Hash Key Design

```javascript
// DynamoDB partition key strategies
class DynamoDBPartitionStrategy {
    constructor(tableConfig) {
        this.tableName = tableConfig.tableName;
        this.partitions = tableConfig.partitions || 10;
        this.items = new Map();
    }
    
    // Composite key design
    createCompositeKey(type, id, timestamp = null) {
        if (timestamp) {
            // Time-based partitioning for even distribution
            const timeBucket = Math.floor(timestamp / (3600 * 1000)); // Hour buckets
            return `${type}#${timeBucket}#${id}`;
        }
        return `${type}#${id}`;
    }
    
    // Sharding strategy for hot partitions
    getShardedKey(baseKey, shardCount = 10) {
        const shard = Math.floor(Math.random() * shardCount);
        return `${baseKey}#${shard}`;
    }
    
    // Write with automatic sharding
    putItem(item) {
        const { pk, sk, ...data } = item;
        
        // Check if this is a hot key
        if (this.isHotKey(pk)) {
            const shardedPk = this.getShardedKey(pk);
            this.items.set(`${shardedPk}#${sk}`, { pk: shardedPk, sk, ...data });
        } else {
            this.items.set(`${pk}#${sk}`, item);
        }
    }
    
    // Query with scatter-gather for sharded keys
    query(pk, skPrefix = null) {
        const results = [];
        
        // Check if key is sharded
        if (this.isHotKey(pk)) {
            // Scatter-gather across shards
            for (let shard = 0; shard < 10; shard++) {
                const shardedPk = `${pk}#${shard}`;
                results.push(...this.queryPartition(shardedPk, skPrefix));
            }
        } else {
            results.push(...this.queryPartition(pk, skPrefix));
        }
        
        return results;
    }
    
    queryPartition(pk, skPrefix) {
        const results = [];
        
        for (const [key, item] of this.items) {
            if (item.pk === pk) {
                if (!skPrefix || item.sk.startsWith(skPrefix)) {
                    results.push(item);
                }
            }
        }
        
        return results;
    }
    
    // Identify hot keys based on access patterns
    isHotKey(key) {
        // Simplified - in reality would track access metrics
        const hotKeys = ['popular_item', 'trending', 'homepage'];
        return hotKeys.some(hot => key.includes(hot));
    }
    
    // GSI design for alternative access patterns
    createGSI(attributeName) {
        const gsi = new Map();
        
        for (const [key, item] of this.items) {
            if (item[attributeName]) {
                const gsiKey = item[attributeName];
                if (!gsi.has(gsiKey)) {
                    gsi.set(gsiKey, []);
                }
                gsi.get(gsiKey).push(item);
            }
        }
        
        return gsi;
    }
}

// Usage
const dynamo = new DynamoDBPartitionStrategy({ tableName: 'Users' });

// Composite keys for hierarchical data
dynamo.putItem({
    pk: dynamo.createCompositeKey('ORG', 'amazon'),
    sk: dynamo.createCompositeKey('USER', 'john@example.com'),
    name: 'John Doe',
    role: 'Engineer'
});

// Time-series data with even distribution
dynamo.putItem({
    pk: dynamo.createCompositeKey('SENSOR', 'temp-001', Date.now()),
    sk: new Date().toISOString(),
    value: 25.5,
    unit: 'celsius'
});
```

### 3. CloudFront Cache Key Optimization

```javascript
// CloudFront cache key and invalidation strategy
class CloudFrontCacheStrategy {
    constructor() {
        this.cache = new Map();
        this.metadata = new Map();
    }
    
    // Generate cache key based on request attributes
    generateCacheKey(request) {
        const {
            uri,
            queryString = {},
            headers = {},
            cookies = {}
        } = request;
        
        // Include whitelisted parameters only
        const whitelistedParams = ['page', 'sort', 'filter'];
        const relevantParams = {};
        for (const param of whitelistedParams) {
            if (queryString[param]) {
                relevantParams[param] = queryString[param];
            }
        }
        
        // Include device type from User-Agent
        const deviceType = this.getDeviceType(headers['User-Agent']);
        
        // Include geo location
        const geoLocation = headers['CloudFront-Viewer-Country'] || 'US';
        
        // Build cache key
        const paramString = Object.entries(relevantParams)
            .sort(([a], [b]) => a.localeCompare(b))
            .map(([k, v]) => `${k}=${v}`)
            .join('&');
            
        return `${uri}#${deviceType}#${geoLocation}#${paramString}`;
    }
    
    getDeviceType(userAgent = '') {
        if (/mobile/i.test(userAgent)) return 'mobile';
        if (/tablet/i.test(userAgent)) return 'tablet';
        return 'desktop';
    }
    
    // Cache with TTL based on content type
    set(request, response) {
        const cacheKey = this.generateCacheKey(request);
        const ttl = this.getTTL(response.contentType);
        
        this.cache.set(cacheKey, response);
        this.metadata.set(cacheKey, {
            timestamp: Date.now(),
            ttl,
            hitCount: 0
        });
        
        // Schedule cleanup
        setTimeout(() => {
            this.cache.delete(cacheKey);
            this.metadata.delete(cacheKey);
        }, ttl * 1000);
    }
    
    // Get with cache hit tracking
    get(request) {
        const cacheKey = this.generateCacheKey(request);
        
        if (this.cache.has(cacheKey)) {
            const meta = this.metadata.get(cacheKey);
            meta.hitCount++;
            
            // Check if still valid
            if (Date.now() - meta.timestamp < meta.ttl * 1000) {
                return {
                    hit: true,
                    response: this.cache.get(cacheKey),
                    age: Math.floor((Date.now() - meta.timestamp) / 1000)
                };
            }
        }
        
        return { hit: false };
    }
    
    // Intelligent TTL based on content
    getTTL(contentType) {
        const ttlMap = {
            'text/html': 300,           // 5 minutes
            'application/json': 60,     // 1 minute for API
            'image/jpeg': 86400,       // 1 day
            'image/png': 86400,        // 1 day
            'text/css': 3600,          // 1 hour
            'application/javascript': 3600, // 1 hour
            'video/mp4': 604800        // 1 week
        };
        
        return ttlMap[contentType] || 300; // Default 5 minutes
    }
    
    // Invalidation patterns
    invalidate(pattern) {
        const keysToInvalidate = [];
        
        for (const key of this.cache.keys()) {
            if (this.matchesPattern(key, pattern)) {
                keysToInvalidate.push(key);
            }
        }
        
        for (const key of keysToInvalidate) {
            this.cache.delete(key);
            this.metadata.delete(key);
        }
        
        return keysToInvalidate.length;
    }
    
    matchesPattern(key, pattern) {
        // Convert wildcard pattern to regex
        const regex = new RegExp(
            pattern.replace(/\*/g, '.*').replace(/\?/g, '.')
        );
        return regex.test(key);
    }
    
    // Cache statistics
    getStats() {
        let totalHits = 0;
        let totalSize = 0;
        
        for (const [key, meta] of this.metadata) {
            totalHits += meta.hitCount;
            const response = this.cache.get(key);
            totalSize += JSON.stringify(response).length;
        }
        
        return {
            entries: this.cache.size,
            totalHits,
            totalSize,
            avgHitRate: this.cache.size > 0 ? totalHits / this.cache.size : 0
        };
    }
}

// Usage
const cdn = new CloudFrontCacheStrategy();

// Cache API response
cdn.set(
    {
        uri: '/api/products',
        queryString: { page: '1', sort: 'price' },
        headers: { 'User-Agent': 'Mozilla... mobile' }
    },
    {
        contentType: 'application/json',
        data: { products: [] }
    }
);

// Invalidate product pages
cdn.invalidate('/api/products*'); // Invalidates all product API calls
```

## Practice Problems

### Problem 1: Two Sum (Using Hash Map)
```javascript
/**
 * Find two numbers that sum to target
 * Time: O(n), Space: O(n)
 */
function twoSum(nums, target) {
    const seen = new Map();
    
    for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];
        
        if (seen.has(complement)) {
            return [seen.get(complement), i];
        }
        
        seen.set(nums[i], i);
    }
    
    return [];
}

// Test
console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
```

### Problem 2: Group Anagrams
```javascript
/**
 * Group anagrams together
 * Time: O(n * k log k), Space: O(n * k)
 * where n is number of strings, k is max length
 */
function groupAnagrams(strs) {
    const map = new Map();
    
    for (const str of strs) {
        // Sort characters to create key
        const key = str.split('').sort().join('');
        
        if (!map.has(key)) {
            map.set(key, []);
        }
        
        map.get(key).push(str);
    }
    
    return Array.from(map.values());
}

// Alternative using character count
function groupAnagramsOptimal(strs) {
    const map = new Map();
    
    for (const str of strs) {
        const count = new Array(26).fill(0);
        
        for (const char of str) {
            count[char.charCodeAt(0) - 97]++;
        }
        
        const key = count.join('#');
        
        if (!map.has(key)) {
            map.set(key, []);
        }
        
        map.get(key).push(str);
    }
    
    return Array.from(map.values());
}

// Test
console.log(groupAnagrams(["eat", "tea", "tan", "ate", "nat", "bat"]));
// [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

### Problem 3: LFU Cache
```javascript
/**
 * Least Frequently Used Cache
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

### Problem 4: Find Duplicate Subtrees
```javascript
/**
 * Find duplicate subtrees in binary tree
 * Time: O(n), Space: O(n)
 */
function findDuplicateSubtrees(root) {
    const result = [];
    const map = new Map();
    
    function serialize(node) {
        if (!node) return '#';
        
        const subtree = `${node.val},${serialize(node.left)},${serialize(node.right)}`;
        
        const count = (map.get(subtree) || 0) + 1;
        map.set(subtree, count);
        
        if (count === 2) {
            result.push(node);
        }
        
        return subtree;
    }
    
    serialize(root);
    return result;
}
```

## Interview Tips

### Common Questions
1. **"How do you handle hash collisions?"**
   - Chaining: Use linked lists or arrays at each slot
   - Open addressing: Linear probing, quadratic probing, double hashing
   - Dynamic resizing when load factor exceeds threshold

2. **"What makes a good hash function?"**
   - Deterministic (same input → same output)
   - Uniform distribution
   - Fast to compute
   - Minimize collisions

3. **"When would you use a Set vs Array?"**
   - Set: Unique values, O(1) membership testing
   - Array: Ordered, indexed access, duplicates allowed

### Red Flags to Avoid
- ❌ Not considering hash collision handling
- ❌ Forgetting about load factor and resizing
- ❌ Using objects as hash map keys without proper serialization
- ❌ Not handling the case when hash table is full (open addressing)

### AWS-Specific Follow-ups
- "Design a distributed cache like ElastiCache"
- "How would you implement cache warming strategies?"
- "Explain consistent hashing for DynamoDB partitions"
- "Design a rate limiter using hash tables"

## Quiz Questions (With Answers)

### Q1: Why is hash table lookup O(1) average case but O(n) worst case?

**Answer:** 
- **O(1) Average Case**: With a good hash function and proper load factor, keys distribute evenly across buckets. Direct array access to bucket + small chain traversal = constant time.
- **O(n) Worst Case**: If all keys hash to the same bucket (poor hash function or adversarial input), it becomes a linked list search.

Example of worst case:
```javascript
// Bad hash function
function badHash(key) {
    return 0; // All keys go to bucket 0!
}
// Now all operations are O(n) as we search through one long chain
```

**Prevention**: Use cryptographic hash functions for security-critical applications, maintain load factor < 0.75, and implement dynamic resizing.

### Q2: Explain the difference between chaining and open addressing for collision resolution.

**Answer:**

**Chaining:**
- Each bucket contains a linked list/array of collided items
- Can store unlimited items (memory permitting)
- Better for high load factors
- Cache performance worse (pointer chasing)
- Simple deletion

**Open Addressing:**
- Find next available slot using probing sequence
- Table size limits capacity
- Better cache locality (contiguous memory)
- Complex deletion (needs tombstones or rehashing)
- Types: Linear probing, quadratic probing, double hashing

```javascript
// Chaining: bucket[2] → [(k1,v1)] → [(k2,v2)] → [(k3,v3)]
// Open Addressing: If bucket[2] full, try bucket[3], then bucket[4]...
```

**AWS Example**: DynamoDB uses consistent hashing with virtual nodes (similar to chaining concept) for partition distribution.

### Q3: What is load factor and why is 0.75 commonly used as the threshold?

**Answer:** Load factor = (number of entries) / (bucket array size)

**Why 0.75?**
- **Mathematical optimization**: Balances space vs time
- **Probability theory**: At 0.75, expected probes for successful search ≈ 1.5
- **Collision probability**: At 0.75, about 50% chance of collision
- **Memory vs Performance trade-off**: 
  - Lower (0.5): Wastes memory but fewer collisions
  - Higher (0.9): Saves memory but more collisions

```javascript
// With 0.75 load factor:
// 10 buckets → resize after 7-8 entries
// Probability of collision ≈ 1 - e^(-0.75) ≈ 0.47
```

Java HashMap uses 0.75, Python dict resizes at 2/3, and Go maps grow at various thresholds.

### Q4: How does consistent hashing solve the rehashing problem in distributed systems?

**Answer:** Traditional hashing requires rehashing all keys when nodes change:
```
Before: hash(key) % 3 servers = server_index
After adding server: hash(key) % 4 servers = different_index!
```

**Consistent Hashing Solution:**
1. Hash servers and keys to same circular space (0 to 2^32)
2. Key belongs to first server clockwise from key's hash
3. Adding/removing server only affects adjacent key ranges

**Benefits:**
- Only K/N keys move when adding server (K=keys, N=servers)
- Virtual nodes provide better distribution
- Used by: Cassandra, DynamoDB, Memcached

```javascript
// Only keys between removed_server and next_server move
Ring: [Server1:100] → [Server2:200] → [Server3:300]
Key(150) → Server2
Remove Server2: Key(150) → Server3 (only keys 100-200 affected)
```

### Q5: What's the difference between a Set and a Map, and when would you use each?

**Answer:**

**Map (Dictionary/Hash Map):**
- Stores key-value pairs
- Use when you need to associate data
- Examples: Caching, counting, lookups

**Set (Hash Set):**
- Stores unique values only
- Use when you only care about membership
- Examples: Deduplication, visited tracking

```javascript
// Map Use Case: Word frequency
const wordCount = new Map();
words.forEach(word => {
    wordCount.set(word, (wordCount.get(word) || 0) + 1);
});

// Set Use Case: Finding unique values
const unique = new Set(arrayWithDuplicates);

// Set Use Case: Tracking visited nodes in graph
const visited = new Set();
if (!visited.has(node)) {
    visited.add(node);
    // process node
}
```

**AWS Example**: CloudFront uses Sets for origin request policies (unique headers) and Maps for cache behaviors (path pattern → behavior).

## Common Mistakes to Avoid

1. **Using Non-Primitive Keys Without Proper Hashing**
   ```javascript
   // BAD: Objects as keys without serialization
   const map = new Map();
   map.set({id: 1}, "value"); // Can't retrieve without same reference!
   
   // GOOD: Serialize objects or use primitive keys
   map.set(JSON.stringify({id: 1}), "value");
   ```

2. **Not Handling Hash Collisions in Custom Implementation**
   ```javascript
   // BAD: Overwriting on collision
   this.buckets[hash] = value;
   
   // GOOD: Chain or probe
   if (!this.buckets[hash]) this.buckets[hash] = [];
   this.buckets[hash].push([key, value]);
   ```

3. **Ignoring Load Factor Management**
   - Always resize when load factor exceeds threshold
   - Not resizing leads to O(n) performance degradation

4. **Using Mutable Keys**
   ```javascript
   // BAD: Key that can change after insertion
   const key = [1, 2, 3];
   map.set(key, "value");
   key.push(4); // Hash changed, can't find entry!
   ```

## Key Takeaways
1. **Hash tables** provide O(1) average case for basic operations
2. **Load factor** management is crucial for performance
3. **Consistent hashing** enables efficient distributed systems
4. **Caching strategies** (LRU, LFU, TTL) optimize system performance
5. **AWS services** heavily rely on hash-based structures for scalability
6. **Choose appropriate collision resolution** based on use case
7. **Consider cache locality** for performance-critical applications

---

*Previous: [← Trees and Graphs](./02-trees-and-graphs.md)*
*Next: [Queues and Stacks →](./04-queues-and-stacks.md)*
