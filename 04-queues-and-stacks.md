# Queues and Stacks

## Overview
Queues and stacks are fundamental linear data structures that control the order of element access. They are essential for many algorithms and system designs, particularly in task scheduling, expression evaluation, and message processing.

### Key Concepts
- **Stack (LIFO)**: Last In, First Out - like a stack of plates
- **Queue (FIFO)**: First In, First Out - like a line at a store
- **Priority Queue**: Elements processed by priority, not order
- **AWS Applications**: SQS, Kinesis, Lambda event processing, CloudFormation stacks

## Stack Implementation

### Array-Based Stack

```javascript
class Stack {
    constructor() {
        this.items = [];  // Using dynamic array as underlying structure
    }
    
    // ============= PUSH OPERATION =============
    // Time: O(1) amortized - WHY?
    // Array push adds to end: O(1) most of the time
    // Occasionally resizes array (doubles capacity): O(n)
    // But resize happens rarely (at powers of 2)
    // Amortized: Total cost/operations = O(1)
    push(element) {
        this.items.push(element);  // Add to top of stack (end of array)
    }
    
    // ============= POP OPERATION =============
    // Time: O(1) - WHY?
    // Removes from end of array - no shifting needed
    // Just decrements length and returns last element
    // No other elements affected
    pop() {
        if (this.isEmpty()) {
            throw new Error('Stack is empty');
        }
        return this.items.pop();  // Remove from top (end of array)
    }
    
    // ============= PEEK OPERATION =============
    // Time: O(1) - WHY?
    // Direct array access by index
    // No modification, just reading
    peek() {
        if (this.isEmpty()) {
            throw new Error('Stack is empty');
        }
        // Access last element without removing
        return this.items[this.items.length - 1];
    }
    
    // ============= UTILITY OPERATIONS =============
    // All O(1) - simple property access or assignment
    isEmpty() {
        return this.items.length === 0;  // Just checking length property
    }
    
    size() {
        return this.items.length;  // Length is maintained by array
    }
    
    // Time: O(1) technically - WHY?
    // Creates new empty array, old one garbage collected
    // Not O(n) because we don't iterate through elements
    clear() {
        this.items = [];  // Replace with new array
    }
}

// Usage
const stack = new Stack();
stack.push(1);
stack.push(2);
stack.push(3);
console.log(stack.pop()); // 3
console.log(stack.peek()); // 2
```

### Linked List Stack

```javascript
class StackNode {
    constructor(value) {
        this.value = value;
        this.next = null;  // Points to node below in stack
    }
}

class LinkedStack {
    constructor() {
        this.top = null;  // Top of stack
        this.size = 0;    // Track size explicitly
    }
    
    // ============= PUSH OPERATION =============
    // Time: O(1) - WHY?
    // Only updating pointers at head
    // No traversal needed, no array resizing
    // Space: O(1) per operation
    push(value) {
        const newNode = new StackNode(value);
        // New node points to current top
        newNode.next = this.top;
        // New node becomes top
        this.top = newNode;
        this.size++;
    }
    
    // ============= POP OPERATION =============
    // Time: O(1) - WHY?
    // Only accessing and updating top pointer
    // Old top becomes eligible for garbage collection
    pop() {
        if (!this.top) {
            throw new Error('Stack is empty');
        }
        const value = this.top.value;
        // Move top pointer to next node
        this.top = this.top.next;  // O(1) pointer update
        this.size--;
        return value;
    }
    
    // ============= PEEK OPERATION =============
    // Time: O(1) - Direct access to top node
    peek() {
        if (!this.top) {
            throw new Error('Stack is empty');
        }
        return this.top.value;  // Just reading, not modifying
    }
    
    // Time: O(1) - Simple null check
    isEmpty() {
        return this.top === null;
    }
}

// WHY USE LINKED STACK VS ARRAY STACK?
// Linked Stack Pros:
// - True O(1) push (no amortization)
// - No wasted space (array might have unused capacity)
// - No size limit (except available memory)
// Linked Stack Cons:
// - Extra memory for pointers
// - Worse cache locality (nodes scattered in memory)
// - No index-based access
```

### Min Stack (O(1) Get Minimum)

```javascript
class MinStack {
    constructor() {
        this.stack = [];     // Main stack for all values
        this.minStack = [];  // Auxiliary stack tracking minimums
    }
    
    // ============= PUSH WITH MIN TRACKING =============
    // Time: O(1) - WHY?
    // Two array pushes, both O(1) amortized
    // Comparison is O(1)
    push(value) {
        // Always push to main stack
        this.stack.push(value);
        
        // Push to minStack if it's new minimum or equal
        // WHY <= and not just <?
        // Need to handle duplicates of minimum value
        // If we pop one minimum, another should still be tracked
        if (this.minStack.length === 0 || value <= this.getMin()) {
            this.minStack.push(value);
        }
    }
    
    // ============= POP WITH MIN TRACKING =============
    // Time: O(1) - Two potential pops, both O(1)
    pop() {
        const value = this.stack.pop();
        
        // If popping the minimum, update minStack
        // This maintains correct minimum for remaining elements
        if (value === this.getMin()) {
            this.minStack.pop();
        }
        
        return value;
    }
    
    // ============= CONSTANT TIME OPERATIONS =============
    // O(1) - Direct array access
    top() {
        return this.stack[this.stack.length - 1];
    }
    
    // O(1) - Direct array access to tracked minimum
    // This is the KEY INNOVATION - WHY IT WORKS:
    // minStack top always contains minimum of all current elements
    // When we pop minimum, next minStack top is next minimum
    getMin() {
        return this.minStack[this.minStack.length - 1];
    }
}

// SPACE COMPLEXITY: O(n) worst case - WHY?
// If elements pushed in descending order,
// every element goes into minStack
// Example: push(5,4,3,2,1) → minStack: [5,4,3,2,1]

// Usage
const minStack = new MinStack();
minStack.push(5);
minStack.push(2);
minStack.push(7);
minStack.push(1);
console.log(minStack.getMin()); // 1
minStack.pop();
console.log(minStack.getMin()); // 2
```

## Queue Implementation

### Array-Based Queue (Circular Buffer)

```javascript
class CircularQueue {
    constructor(capacity = 10) {
        this.items = new Array(capacity);  // Fixed size array
        this.capacity = capacity;
        this.front = 0;      // Index of first element
        this.rear = -1;      // Index of last element (-1 means empty)
        this.size = 0;       // Current number of elements
    }
    
    // ============= ENQUEUE OPERATION =============
    // Time: O(1) - WHY?
    // Direct array access with modulo arithmetic
    // No shifting of elements needed
    // Space: O(1) - no new memory allocated
    enqueue(element) {
        if (this.isFull()) {
            throw new Error('Queue is full');
        }
        
        // WHY MODULO? Wraps around to beginning when reaching end
        // Example: capacity=5, rear=4, (4+1)%5 = 0 (wraps to index 0)
        // This is what makes it "circular" - reuses array positions
        this.rear = (this.rear + 1) % this.capacity;
        this.items[this.rear] = element;
        this.size++;
    }
    
    // ============= DEQUEUE OPERATION =============
    // Time: O(1) - WHY?
    // Unlike array.shift() which is O(n) due to shifting
    // We just move the front pointer - no element movement
    // This is the KEY ADVANTAGE of circular buffer
    dequeue() {
        if (this.isEmpty()) {
            throw new Error('Queue is empty');
        }
        
        const element = this.items[this.front];
        this.items[this.front] = null; // WHY SET NULL?
        // Helps garbage collector - removes reference
        // Prevents memory leaks with object references
        
        // Move front pointer circularly
        // Example: capacity=5, front=4, (4+1)%5 = 0 (wraps to beginning)
        this.front = (this.front + 1) % this.capacity;
        this.size--;
        return element;
    }
    
    // ============= PEEK OPERATION =============
    // Time: O(1) - Direct array access by index
    peek() {
        if (this.isEmpty()) {
            throw new Error('Queue is empty');
        }
        return this.items[this.front];  // Just read, don't modify
    }
    
    // ============= UTILITY OPERATIONS =============
    // All O(1) - simple comparisons or property access
    isEmpty() {
        return this.size === 0;  // Track size explicitly for O(1)
        // Alternative: check if front and rear meet with size tracking
    }
    
    isFull() {
        return this.size === this.capacity;
        // Without size: would need complex circular arithmetic
    }
}

// WHY USE CIRCULAR QUEUE?
// 1. Fixed memory - good for embedded systems
// 2. O(1) dequeue vs O(n) for array.shift()
// 3. No memory allocation during operations
// 4. Cache-friendly - contiguous memory
// Drawback: Fixed capacity - can't grow dynamically
```

### Linked List Queue

```javascript
class QueueNode {
    constructor(value) {
        this.value = value;
        this.next = null;
    }
}

class LinkedQueue {
    constructor() {
        this.front = null;
        this.rear = null;
        this.size = 0;
    }
    
    // O(1)
    enqueue(value) {
        const newNode = new QueueNode(value);
        
        if (!this.rear) {
            this.front = newNode;
            this.rear = newNode;
        } else {
            this.rear.next = newNode;
            this.rear = newNode;
        }
        
        this.size++;
    }
    
    // O(1)
    dequeue() {
        if (!this.front) {
            throw new Error('Queue is empty');
        }
        
        const value = this.front.value;
        this.front = this.front.next;
        
        if (!this.front) {
            this.rear = null;
        }
        
        this.size--;
        return value;
    }
    
    // O(1)
    peek() {
        if (!this.front) {
            throw new Error('Queue is empty');
        }
        return this.front.value;
    }
    
    // O(1)
    isEmpty() {
        return this.front === null;
    }
}
```

### Deque (Double-Ended Queue)

```javascript
class Deque {
    constructor() {
        this.items = [];
    }
    
    // O(1)
    addFront(element) {
        this.items.unshift(element);
    }
    
    // O(1)
    addRear(element) {
        this.items.push(element);
    }
    
    // O(1)
    removeFront() {
        if (this.isEmpty()) {
            throw new Error('Deque is empty');
        }
        return this.items.shift();
    }
    
    // O(1)
    removeRear() {
        if (this.isEmpty()) {
            throw new Error('Deque is empty');
        }
        return this.items.pop();
    }
    
    // O(1)
    peekFront() {
        if (this.isEmpty()) {
            throw new Error('Deque is empty');
        }
        return this.items[0];
    }
    
    // O(1)
    peekRear() {
        if (this.isEmpty()) {
            throw new Error('Deque is empty');
        }
        return this.items[this.items.length - 1];
    }
    
    // O(1)
    isEmpty() {
        return this.items.length === 0;
    }
    
    // O(1)
    size() {
        return this.items.length;
    }
}

// Usage - Sliding Window Maximum
function maxSlidingWindow(nums, k) {
    const deque = new Deque();
    const result = [];
    
    for (let i = 0; i < nums.length; i++) {
        // Remove elements outside window
        while (!deque.isEmpty() && deque.peekFront() < i - k + 1) {
            deque.removeFront();
        }
        
        // Remove smaller elements
        while (!deque.isEmpty() && nums[deque.peekRear()] < nums[i]) {
            deque.removeRear();
        }
        
        deque.addRear(i);
        
        if (i >= k - 1) {
            result.push(nums[deque.peekFront()]);
        }
    }
    
    return result;
}
```

## Priority Queue (Heap)

### Binary Heap Implementation

```javascript
class PriorityQueue {
    constructor(compareFn = (a, b) => a < b) {
        this.heap = [];
        this.compare = compareFn; // Min heap by default (a < b)
        // For max heap: use (a, b) => a > b
    }
    
    // ============= ENQUEUE (INSERT) =============
    // Time: O(log n) - WHY?
    // Insert at end: O(1)
    // Bubble up: O(log n) - traverse height of tree
    // Height of complete binary tree = log n
    enqueue(value, priority) {
        const node = { value, priority };
        this.heap.push(node);  // Add to end - maintains complete tree
        this.bubbleUp(this.heap.length - 1);  // Restore heap property
    }
    
    // ============= DEQUEUE (EXTRACT MIN/MAX) =============
    // Time: O(log n) - WHY?
    // Remove root: O(1)
    // Move last to root: O(1)
    // Bubble down: O(log n) - traverse height of tree
    dequeue() {
        if (this.isEmpty()) {
            throw new Error('Priority queue is empty');
        }
        
        const min = this.heap[0];  // Save root (min/max element)
        const end = this.heap.pop();  // Remove last element
        
        // If heap still has elements, put last element at root
        // and restore heap property
        if (this.heap.length > 0) {
            this.heap[0] = end;
            this.bubbleDown(0);  // Restore heap property from root
        }
        
        return min.value;
    }
    
    // ============= BUBBLE UP (HEAPIFY UP) =============
    // Time: O(log n) - WHY?
    // Worst case: element bubbles from leaf to root
    // Path length = tree height = log n
    // Each step: O(1) comparison and swap
    bubbleUp(index) {
        const element = this.heap[index];
        
        // Continue until we reach root or heap property satisfied
        while (index > 0) {
            // PARENT INDEX FORMULA: (i-1)/2 - WHY?
            // In array representation of complete binary tree:
            // Left child of i: 2i+1
            // Right child of i: 2i+2
            // Parent of i: floor((i-1)/2)
            const parentIndex = Math.floor((index - 1) / 2);
            const parent = this.heap[parentIndex];
            
            // Check if heap property satisfied
            // For min heap: child >= parent
            // For max heap: child <= parent
            if (!this.compare(element.priority, parent.priority)) break;
            
            // Move parent down (don't swap yet - optimization)
            this.heap[index] = parent;
            index = parentIndex;
        }
        
        // Place element at final position
        this.heap[index] = element;
    }
    
    // ============= BUBBLE DOWN (HEAPIFY DOWN) =============
    // Time: O(log n) - WHY?
    // Worst case: element sinks from root to leaf
    // Path length = tree height = log n
    bubbleDown(index) {
        const length = this.heap.length;
        const element = this.heap[index];
        
        // Continue until we reach a leaf or heap property satisfied
        while (true) {
            // CHILDREN INDEX FORMULAS - WHY?
            // Left child: 2i+1 (skip all nodes before parent's level)
            // Right child: 2i+2 (one more than left)
            let leftChildIdx = 2 * index + 1;
            let rightChildIdx = 2 * index + 2;
            let swapIdx = null;  // Track which child to swap with
            
            // Check left child
            if (leftChildIdx < length) {
                const leftChild = this.heap[leftChildIdx];
                // For min heap: if child < parent, mark for swap
                if (this.compare(leftChild.priority, element.priority)) {
                    swapIdx = leftChildIdx;
                }
            }
            
            // Check right child
            if (rightChildIdx < length) {
                const rightChild = this.heap[rightChildIdx];
                // Compare with better candidate (left child or parent)
                const compareElement = swapIdx === null ? element : this.heap[swapIdx];
                // For min heap: choose smaller child
                if (this.compare(rightChild.priority, compareElement.priority)) {
                    swapIdx = rightChildIdx;
                }
            }
            
            // If no swap needed, heap property satisfied
            if (swapIdx === null) break;
            
            // Move child up (don't swap yet - optimization)
            this.heap[index] = this.heap[swapIdx];
            index = swapIdx;
        }
        
        // Place element at final position
        this.heap[index] = element;
    }
    
    // ============= PEEK OPERATION =============
    // Time: O(1) - Root always contains min/max
    // This is the power of heap - constant time access to extreme
    peek() {
        if (this.isEmpty()) {
            throw new Error('Priority queue is empty');
        }
        return this.heap[0].value;  // Root has highest priority
    }
    
    // O(1) - Simple property checks
    isEmpty() {
        return this.heap.length === 0;
    }
    
    size() {
        return this.heap.length;
    }
}

// WHY USE HEAP FOR PRIORITY QUEUE?
// 1. O(log n) insert and extract vs O(n) for sorted array insert
// 2. O(1) peek vs O(n) for unsorted array
// 3. Complete tree - efficient array representation
// 4. No need to maintain full sorting - only heap property
```

// Usage - Task Scheduler
const tasks = new PriorityQueue();
tasks.enqueue('Low priority task', 3);
tasks.enqueue('High priority task', 1);
tasks.enqueue('Medium priority task', 2);

console.log(tasks.dequeue()); // High priority task
console.log(tasks.dequeue()); // Medium priority task
console.log(tasks.dequeue()); // Low priority task
```

## AWS Applications

### 1. SQS (Simple Queue Service) Simulator

```javascript
class SQSSimulator {
    constructor(config = {}) {
        this.queues = new Map();
        this.dlq = new Map(); // Dead Letter Queue
        this.visibilityTimeout = config.visibilityTimeout || 30000; // 30 seconds
        this.maxReceiveCount = config.maxReceiveCount || 3;
        this.messageRetention = config.messageRetention || 345600000; // 4 days
    }
    
    createQueue(queueName, isFIFO = false) {
        const queue = {
            name: queueName,
            messages: [],
            isFIFO,
            inFlight: new Map(),
            attributes: {
                createdTimestamp: Date.now(),
                approximateNumberOfMessages: 0,
                approximateNumberOfMessagesNotVisible: 0
            }
        };
        
        this.queues.set(queueName, queue);
        return queue;
    }
    
    // Send message to queue
    sendMessage(queueName, messageBody, attributes = {}) {
        const queue = this.queues.get(queueName);
        if (!queue) throw new Error(`Queue ${queueName} not found`);
        
        const message = {
            messageId: this.generateMessageId(),
            body: messageBody,
            attributes,
            receiveCount: 0,
            sentTimestamp: Date.now(),
            messageGroupId: attributes.messageGroupId || null,
            messageDeduplicationId: attributes.messageDeduplicationId || null
        };
        
        // FIFO queue deduplication
        if (queue.isFIFO && message.messageDeduplicationId) {
            const isDuplicate = queue.messages.some(m => 
                m.messageDeduplicationId === message.messageDeduplicationId &&
                Date.now() - m.sentTimestamp < 300000 // 5 min dedup window
            );
            
            if (isDuplicate) {
                return { messageId: null, duplicate: true };
            }
        }
        
        queue.messages.push(message);
        queue.attributes.approximateNumberOfMessages++;
        
        return { messageId: message.messageId, duplicate: false };
    }
    
    // Receive messages from queue
    receiveMessage(queueName, maxMessages = 1, waitTime = 0) {
        const queue = this.queues.get(queueName);
        if (!queue) throw new Error(`Queue ${queueName} not found`);
        
        const messages = [];
        const now = Date.now();
        
        // Long polling simulation
        const pollEndTime = now + (waitTime * 1000);
        
        while (messages.length < maxMessages && Date.now() < pollEndTime) {
            // Find available messages
            for (let i = 0; i < queue.messages.length && messages.length < maxMessages; i++) {
                const message = queue.messages[i];
                
                // Check if message is visible
                if (!queue.inFlight.has(message.messageId)) {
                    // Move to in-flight
                    queue.inFlight.set(message.messageId, {
                        message,
                        visibilityTimeout: now + this.visibilityTimeout,
                        receiptHandle: this.generateReceiptHandle()
                    });
                    
                    message.receiveCount++;
                    messages.push({
                        messageId: message.messageId,
                        body: message.body,
                        attributes: message.attributes,
                        receiptHandle: queue.inFlight.get(message.messageId).receiptHandle
                    });
                    
                    // Remove from main queue
                    queue.messages.splice(i, 1);
                    i--;
                }
            }
            
            if (messages.length === 0 && waitTime > 0) {
                // Simulate waiting
                break; // In real implementation, would actually wait
            } else {
                break;
            }
        }
        
        queue.attributes.approximateNumberOfMessages = queue.messages.length;
        queue.attributes.approximateNumberOfMessagesNotVisible = queue.inFlight.size;
        
        return messages;
    }
    
    // Delete message from queue
    deleteMessage(queueName, receiptHandle) {
        const queue = this.queues.get(queueName);
        if (!queue) throw new Error(`Queue ${queueName} not found`);
        
        for (const [messageId, inFlightData] of queue.inFlight) {
            if (inFlightData.receiptHandle === receiptHandle) {
                queue.inFlight.delete(messageId);
                queue.attributes.approximateNumberOfMessagesNotVisible--;
                return true;
            }
        }
        
        return false;
    }
    
    // Change message visibility timeout
    changeMessageVisibility(queueName, receiptHandle, visibilityTimeout) {
        const queue = this.queues.get(queueName);
        if (!queue) throw new Error(`Queue ${queueName} not found`);
        
        for (const inFlightData of queue.inFlight.values()) {
            if (inFlightData.receiptHandle === receiptHandle) {
                inFlightData.visibilityTimeout = Date.now() + (visibilityTimeout * 1000);
                return true;
            }
        }
        
        return false;
    }
    
    // Process visibility timeouts
    processTimeouts() {
        const now = Date.now();
        
        for (const queue of this.queues.values()) {
            for (const [messageId, inFlightData] of queue.inFlight) {
                if (inFlightData.visibilityTimeout <= now) {
                    const message = inFlightData.message;
                    
                    // Check max receive count for DLQ
                    if (message.receiveCount >= this.maxReceiveCount) {
                        // Move to DLQ
                        if (!this.dlq.has(queue.name)) {
                            this.dlq.set(queue.name, []);
                        }
                        this.dlq.get(queue.name).push(message);
                    } else {
                        // Return to queue
                        queue.messages.push(message);
                        queue.attributes.approximateNumberOfMessages++;
                    }
                    
                    queue.inFlight.delete(messageId);
                    queue.attributes.approximateNumberOfMessagesNotVisible--;
                }
            }
        }
    }
    
    generateMessageId() {
        return `msg-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    }
    
    generateReceiptHandle() {
        return `receipt-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    }
}

// Usage
const sqs = new SQSSimulator();
sqs.createQueue('order-processing');

// Send messages
sqs.sendMessage('order-processing', JSON.stringify({ orderId: '123', amount: 99.99 }));
sqs.sendMessage('order-processing', JSON.stringify({ orderId: '124', amount: 149.99 }));

// Receive and process messages
const messages = sqs.receiveMessage('order-processing', 10);
for (const message of messages) {
    console.log('Processing:', message.body);
    // Process message...
    sqs.deleteMessage('order-processing', message.receiptHandle);
}
```

### 2. Kinesis Stream Simulator

```javascript
class KinesisStreamSimulator {
    constructor(streamName, shardCount = 1) {
        this.streamName = streamName;
        this.shards = [];
        this.iterators = new Map();
        
        // Create shards
        for (let i = 0; i < shardCount; i++) {
            this.shards.push({
                shardId: `shard-${i}`,
                records: [],
                sequenceNumberCounter: 0,
                hashKeyRange: {
                    start: Math.floor(i * (2**128 / shardCount)),
                    end: Math.floor((i + 1) * (2**128 / shardCount)) - 1
                }
            });
        }
    }
    
    // Put single record
    putRecord(data, partitionKey) {
        const shard = this.selectShard(partitionKey);
        const sequenceNumber = this.generateSequenceNumber(shard);
        
        const record = {
            data,
            partitionKey,
            sequenceNumber,
            approximateArrivalTimestamp: Date.now(),
            shardId: shard.shardId
        };
        
        shard.records.push(record);
        
        // Trim old records (retention period simulation - keep last 1000)
        if (shard.records.length > 1000) {
            shard.records.shift();
        }
        
        return {
            shardId: shard.shardId,
            sequenceNumber
        };
    }
    
    // Put multiple records
    putRecords(records) {
        const results = [];
        
        for (const record of records) {
            try {
                const result = this.putRecord(record.data, record.partitionKey);
                results.push({
                    ...result,
                    errorCode: null,
                    errorMessage: null
                });
            } catch (error) {
                results.push({
                    errorCode: 'InternalError',
                    errorMessage: error.message
                });
            }
        }
        
        return {
            failedRecordCount: results.filter(r => r.errorCode).length,
            records: results
        };
    }
    
    // Get shard iterator
    getShardIterator(shardId, iteratorType, startingSequenceNumber = null) {
        const shard = this.shards.find(s => s.shardId === shardId);
        if (!shard) throw new Error(`Shard ${shardId} not found`);
        
        let position = 0;
        
        switch (iteratorType) {
            case 'TRIM_HORIZON':
                position = 0;
                break;
            case 'LATEST':
                position = shard.records.length;
                break;
            case 'AT_SEQUENCE_NUMBER':
            case 'AFTER_SEQUENCE_NUMBER':
                const index = shard.records.findIndex(r => 
                    r.sequenceNumber === startingSequenceNumber
                );
                position = iteratorType === 'AT_SEQUENCE_NUMBER' ? index : index + 1;
                break;
            case 'AT_TIMESTAMP':
                // Find first record after timestamp
                position = shard.records.findIndex(r => 
                    r.approximateArrivalTimestamp >= startingSequenceNumber
                );
                break;
        }
        
        const iteratorId = `iter-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
        this.iterators.set(iteratorId, {
            shardId,
            position,
            expiresAt: Date.now() + 300000 // 5 minutes
        });
        
        return iteratorId;
    }
    
    // Get records from iterator
    getRecords(shardIterator, limit = 1000) {
        const iterator = this.iterators.get(shardIterator);
        if (!iterator) throw new Error('Iterator expired or invalid');
        
        if (Date.now() > iterator.expiresAt) {
            this.iterators.delete(shardIterator);
            throw new Error('Iterator expired');
        }
        
        const shard = this.shards.find(s => s.shardId === iterator.shardId);
        const records = [];
        
        const endPosition = Math.min(
            iterator.position + limit,
            shard.records.length
        );
        
        for (let i = iterator.position; i < endPosition; i++) {
            records.push(shard.records[i]);
        }
        
        // Update iterator position
        iterator.position = endPosition;
        
        // Generate next iterator
        const nextIterator = `iter-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
        this.iterators.set(nextIterator, {
            ...iterator,
            expiresAt: Date.now() + 300000
        });
        
        return {
            records,
            nextShardIterator: nextIterator,
            millisBehindLatest: Math.max(0, shard.records.length - endPosition) * 100
        };
    }
    
    // Select shard based on partition key
    selectShard(partitionKey) {
        const hash = this.hashPartitionKey(partitionKey);
        
        for (const shard of this.shards) {
            if (hash >= shard.hashKeyRange.start && hash <= shard.hashKeyRange.end) {
                return shard;
            }
        }
        
        // Fallback to first shard
        return this.shards[0];
    }
    
    // Simple hash function for partition key
    hashPartitionKey(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
            hash = hash & hash;
        }
        return Math.abs(hash);
    }
    
    generateSequenceNumber(shard) {
        return `${shard.shardId}-${++shard.sequenceNumberCounter}`;
    }
}

// Usage
const kinesis = new KinesisStreamSimulator('clickstream', 2);

// Producer
kinesis.putRecord(
    JSON.stringify({ userId: '123', action: 'click', timestamp: Date.now() }),
    'user-123'
);

// Batch producer
kinesis.putRecords([
    { data: JSON.stringify({ event: 'login' }), partitionKey: 'user-456' },
    { data: JSON.stringify({ event: 'purchase' }), partitionKey: 'user-789' }
]);

// Consumer
const iterator = kinesis.getShardIterator('shard-0', 'TRIM_HORIZON');
const response = kinesis.getRecords(iterator);
console.log('Records:', response.records);
```

### 3. Lambda Event Queue Processing

```javascript
class LambdaEventProcessor {
    constructor(config = {}) {
        this.concurrency = config.concurrency || 10;
        this.batchSize = config.batchSize || 10;
        this.eventQueue = [];
        this.dlq = [];
        this.processing = new Set();
        this.maxRetries = config.maxRetries || 3;
    }
    
    // Add events to queue
    addEvents(events) {
        for (const event of events) {
            this.eventQueue.push({
                ...event,
                retryCount: 0,
                enqueuedAt: Date.now()
            });
        }
    }
    
    // Process events with concurrency control
    async processEvents(handlerFunction) {
        const results = [];
        
        while (this.eventQueue.length > 0 || this.processing.size > 0) {
            // Fill up to concurrency limit
            while (this.processing.size < this.concurrency && this.eventQueue.length > 0) {
                const batch = this.eventQueue.splice(0, this.batchSize);
                const promise = this.processBatch(batch, handlerFunction);
                this.processing.add(promise);
                
                promise.then(result => {
                    results.push(...result);
                    this.processing.delete(promise);
                });
            }
            
            // Wait for at least one to complete
            if (this.processing.size > 0) {
                await Promise.race(this.processing);
            }
        }
        
        return results;
    }
    
    async processBatch(batch, handlerFunction) {
        const results = [];
        
        for (const event of batch) {
            try {
                const result = await handlerFunction(event);
                results.push({
                    eventId: event.id,
                    status: 'success',
                    result
                });
            } catch (error) {
                event.retryCount++;
                
                if (event.retryCount < this.maxRetries) {
                    // Exponential backoff
                    setTimeout(() => {
                        this.eventQueue.push(event);
                    }, Math.pow(2, event.retryCount) * 1000);
                    
                    results.push({
                        eventId: event.id,
                        status: 'retry',
                        retryCount: event.retryCount
                    });
                } else {
                    // Move to DLQ
                    this.dlq.push({
                        ...event,
                        error: error.message,
                        failedAt: Date.now()
                    });
                    
                    results.push({
                        eventId: event.id,
                        status: 'dlq',
                        error: error.message
                    });
                }
            }
        }
        
        return results;
    }
    
    // Get DLQ contents
    getDLQ() {
        return this.dlq;
    }
    
    // Reprocess DLQ
    reprocessDLQ() {
        const events = this.dlq.splice(0);
        for (const event of events) {
            event.retryCount = 0;
            delete event.error;
            delete event.failedAt;
            this.eventQueue.push(event);
        }
        return events.length;
    }
}

// Usage
const processor = new LambdaEventProcessor({
    concurrency: 5,
    batchSize: 25,
    maxRetries: 3
});

// Add events
processor.addEvents([
    { id: '1', type: 'order', data: { orderId: '123' } },
    { id: '2', type: 'payment', data: { amount: 99.99 } },
    // ... more events
]);

// Process with Lambda handler
async function lambdaHandler(event) {
    // Simulate processing
    if (Math.random() < 0.1) {
        throw new Error('Random failure');
    }
    
    await new Promise(resolve => setTimeout(resolve, 100));
    return { processed: true, eventId: event.id };
}

processor.processEvents(lambdaHandler).then(results => {
    console.log('Processing complete:', results);
    console.log('Failed events in DLQ:', processor.getDLQ());
});
```

## Practice Problems

### Problem 1: Valid Parentheses
```javascript
/**
 * Check if parentheses are valid
 * Time: O(n), Space: O(n)
 */
function isValid(s) {
    const stack = [];
    const pairs = {
        '(': ')',
        '[': ']',
        '{': '}'
    };
    
    for (const char of s) {
        if (char in pairs) {
            // Opening bracket
            stack.push(char);
        } else {
            // Closing bracket
            if (stack.length === 0) return false;
            const last = stack.pop();
            if (pairs[last] !== char) return false;
        }
    }
    
    return stack.length === 0;
}

// Test
console.log(isValid("()")); // true
console.log(isValid("()[]{}")); // true
console.log(isValid("(]")); // false
console.log(isValid("([)]")); // false
```

### Problem 2: Implement Queue Using Stacks
```javascript
/**
 * Queue using two stacks
 * Amortized O(1) for all operations
 */
class QueueUsingStacks {
    constructor() {
        this.inStack = [];  // For enqueue
        this.outStack = []; // For dequeue
    }
    
    // O(1)
    push(x) {
        this.inStack.push(x);
    }
    
    // Amortized O(1)
    pop() {
        this.peek(); // Ensures outStack has elements
        return this.outStack.pop();
    }
    
    // Amortized O(1)
    peek() {
        if (this.outStack.length === 0) {
            // Transfer all from inStack to outStack
            while (this.inStack.length > 0) {
                this.outStack.push(this.inStack.pop());
            }
        }
        return this.outStack[this.outStack.length - 1];
    }
    
    // O(1)
    empty() {
        return this.inStack.length === 0 && this.outStack.length === 0;
    }
}
```

### Problem 3: Daily Temperatures (Monotonic Stack)
```javascript
/**
 * Find days until warmer temperature
 * Time: O(n), Space: O(n)
 */
function dailyTemperatures(temperatures) {
    const n = temperatures.length;
    const result = new Array(n).fill(0);
    const stack = []; // Stack of indices
    
    for (let i = 0; i < n; i++) {
        // Pop all temperatures lower than current
        while (stack.length > 0 && temperatures[stack[stack.length - 1]] < temperatures[i]) {
            const prevIndex = stack.pop();
            result[prevIndex] = i - prevIndex;
        }
        stack.push(i);
    }
    
    return result;
}

// Test
console.log(dailyTemperatures([73,74,75,71,69,72,76,73]));
// Output: [1,1,4,2,1,1,0,0]
```

### Problem 4: Evaluate Reverse Polish Notation
```javascript
/**
 * Evaluate expression in RPN
 * Time: O(n), Space: O(n)
 */
function evalRPN(tokens) {
    const stack = [];
    const operators = {
        '+': (a, b) => a + b,
        '-': (a, b) => a - b,
        '*': (a, b) => a * b,
        '/': (a, b) => Math.trunc(a / b)
    };
    
    for (const token of tokens) {
        if (token in operators) {
            const b = stack.pop();
            const a = stack.pop();
            stack.push(operators[token](a, b));
        } else {
            stack.push(parseInt(token));
        }
    }
    
    return stack[0];
}

// Test
console.log(evalRPN(["2","1","+","3","*"])); // 9
console.log(evalRPN(["4","13","5","/","+"])); // 6
```

### Problem 5: Task Scheduler
```javascript
/**
 * Schedule tasks with cooldown period
 * Time: O(n), Space: O(1) - at most 26 tasks
 */
function leastInterval(tasks, n) {
    // Count frequency of each task (A-Z = 26 possible tasks)
    const freq = new Array(26).fill(0);
    for (const task of tasks) {
        // Convert char to index: 'A'=0, 'B'=1, etc.
        freq[task.charCodeAt(0) - 65]++;
    }
    
    // Sort frequencies descending - most frequent first
    freq.sort((a, b) => b - a);
    
    // KEY INSIGHT: Most frequent task determines minimum time
    // WHY? It needs the most cooldown periods between executions
    const maxFreq = freq[0];
    
    // Calculate idle slots needed
    // (maxFreq - 1): Number of gaps between max frequency task
    // * n: Cooldown period for each gap
    let idleSlots = (maxFreq - 1) * n;
    
    // Fill idle slots with other tasks
    for (let i = 1; i < 26 && freq[i] > 0; i++) {
        // Can place at most (maxFreq - 1) of each task in the gaps
        // WHY maxFreq - 1? Last execution doesn't need cooldown after it
        idleSlots -= Math.min(freq[i], maxFreq - 1);
    }
    
    // If we have more tasks than idle slots, no idle time needed
    // All slots are filled with actual tasks
    idleSlots = Math.max(0, idleSlots);
    
    return tasks.length + idleSlots;
}

// Alternative using Priority Queue
function leastIntervalPQ(tasks, n) {
    const freqMap = new Map();
    for (const task of tasks) {
        freqMap.set(task, (freqMap.get(task) || 0) + 1);
    }
    
    // Max heap of frequencies
    const pq = new PriorityQueue((a, b) => a > b);
    for (const freq of freqMap.values()) {
        pq.enqueue(freq, freq);
    }
    
    let time = 0;
    
    while (!pq.isEmpty()) {
        const temp = [];
        let cycle = n + 1;
        
        while (cycle > 0 && !pq.isEmpty()) {
            const freq = pq.dequeue();
            if (freq > 1) {
                temp.push(freq - 1);
            }
            time++;
            cycle--;
        }
        
        // Re-add tasks that still need processing
        for (const freq of temp) {
            pq.enqueue(freq, freq);
        }
        
        // Add idle time if needed
        if (!pq.isEmpty()) {
            time += cycle;
        }
    }
    
    return time;
}
```

## Interview Tips

### Common Questions
1. **"When would you use a Stack vs Queue?"**
   - Stack: Function calls, undo operations, expression evaluation
   - Queue: Task scheduling, BFS, message processing
   - Both have O(1) insertion/removal at their respective ends

2. **"How do you implement a Queue with O(1) operations?"**
   - Use circular buffer with fixed size
   - Use linked list with head and tail pointers
   - Track front/rear indices for array implementation

3. **"Explain priority queue and its applications"**
   - Heap-based implementation gives O(log n) operations
   - Used in: Dijkstra's algorithm, task scheduling, median finding
   - Can be min-heap or max-heap based on needs

### Red Flags to Avoid
- ❌ Using array.shift() for queue dequeue (O(n) operation)
- ❌ Not handling empty stack/queue edge cases
- ❌ Forgetting to update size counters
- ❌ Not considering overflow in fixed-size implementations

### AWS-Specific Follow-ups
- "Design a distributed message queue like SQS"
- "How would you implement DLQ (Dead Letter Queue)?"
- "Explain FIFO vs Standard queues in SQS"
- "Design a rate limiter using queues"

## Key Takeaways
1. **Stacks** are perfect for nested structures and backtracking
2. **Queues** excel at maintaining order and fairness
3. **Priority Queues** optimize task processing by importance
4. **Monotonic stacks/queues** solve range queries efficiently
5. **AWS services** use queues extensively for decoupling and scaling

---

*Previous: [← Hash Maps and Sets](./03-hash-maps-and-sets.md)*
*Next: [Searching and Sorting →](./05-searching-and-sorting.md)*
