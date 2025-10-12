# 🔗 Linked Lists

## Overview
Linked Lists are dynamic data structures where elements (nodes) are connected through pointers. Unlike arrays, they don't require contiguous memory and excel at insertion/deletion operations.

## Key Concepts

### Types of Linked Lists
- **Singly Linked**: Each node points to the next
- **Doubly Linked**: Nodes point to both next and previous
- **Circular**: Last node points back to first

### Common Operations
| Operation | Time Complexity | Description |
|-----------|----------------|-------------|
| Insert at Head | O(1) | Add node at beginning |
| Insert at Tail | O(n) or O(1)* | Add node at end (*with tail pointer) |
| Delete Node | O(n) | Remove specific node |
| Search | O(n) | Find node with value |
| Reverse | O(n) | Reverse entire list |

## Common Patterns
1. **Two Pointers** - Fast/slow pointers for cycle detection
2. **Dummy Head** - Simplify edge cases
3. **Recursion** - Natural for list traversal
4. **In-place Reversal** - Modify pointers without extra space

## Problems in This Section

### Basic Operations
- [Insert Node at Tail](problems/insert-node-at-tail.md) - Fundamental insertion
- [Insert Node at Position](problems/insert-node-at-position.md) - Positional insertion
- [Detect Cycle](problems/detect-cycle.md) - Floyd's algorithm

### Intermediate
- [Merge Two Sorted Lists](problems/merge-two-sorted-lists.md) - Two-pointer merge
- [Reverse Nodes in k-Group](problems/reverse-nodes-in-k-group.md) - Group reversal

### Advanced
- [LRU Cache](problems/lru-cache.md) - Doubly linked list + HashMap

## Tips for Success
1. **Draw diagrams** - Visualize pointer changes
2. **Handle edge cases** - Empty list, single node, cycle
3. **Track previous node** - Often needed for deletion
4. **Consider dummy nodes** - Simplify head operations
5. **Test with examples** - Walk through pointer updates

## Interview Approach
```javascript
// Template for linked list problems
class ListNode {
    constructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

// Common patterns to remember:
// 1. Dummy head for easier manipulation
const dummy = new ListNode(0);
dummy.next = head;

// 2. Two pointers (fast/slow)
let slow = head, fast = head;

// 3. Reverse pattern
let prev = null, curr = head;
while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
}
```

## When to Use Linked Lists
✅ **Good for:**
- Frequent insertion/deletion at arbitrary positions
- Unknown or dynamic size
- Implementation of stacks/queues
- Memory-efficient for sparse data

❌ **Avoid when:**
- Need random access (use arrays)
- Cache performance is critical
- Need to minimize memory per element
