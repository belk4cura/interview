# Reverse Nodes in k-Group

## 🎯 What Are We Trying To Solve?

Reversing a linked list in groups of k is like flipping pages in chunks - you reverse k pages at a time, leaving any remaining pages in their original order.

## 📝 The Problem

Given a linked list, reverse the nodes in groups of k. If the number of nodes is not a multiple of k, leave the remaining nodes as is.

```
Example:
Input: 1→2→3→4→5, k=2
Output: 2→1→4→3→5

Input: 1→2→3→4→5, k=3
Output: 3→2→1→4→5
```

## 💻 The Code

```javascript
class ListNode {
    constructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

/**
 * Reverse Nodes in k-Group
 * Time: O(n), Space: O(1)
 */
function reverseKGroup(head, k) {
    // Check if we have k nodes
    let count = 0;
    let node = head;
    while (node && count < k) {
        node = node.next;
        count++;
    }
    
    if (count < k) return head; // Less than k nodes remaining
    
    // Reverse k nodes
    let curr = head;
    let prev = null;
    let next = null;
    count = 0;
    
    while (curr && count < k) {
        next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
        count++;
    }
    
    // Recursively reverse remaining groups
    if (next) {
        head.next = reverseKGroup(next, k);
    }
    
    return prev; // New head of this group
}

// Iterative approach
function reverseKGroupIterative(head, k) {
    const dummy = new ListNode(0);
    dummy.next = head;
    let prevGroup = dummy;
    
    while (true) {
        // Check if k nodes exist
        let kthNode = prevGroup;
        for (let i = 0; i < k; i++) {
            kthNode = kthNode.next;
            if (!kthNode) return dummy.next;
        }
        
        const nextGroup = kthNode.next;
        
        // Reverse current group
        let prev = nextGroup;
        let curr = prevGroup.next;
        
        for (let i = 0; i < k; i++) {
            const next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        
        const tmp = prevGroup.next;
        prevGroup.next = kthNode;
        prevGroup = tmp;
    }
}
```

## 🎨 Key Insights

- **Group Detection**: Check if k nodes exist before reversing
- **Pointer Management**: Track group boundaries carefully
- **Recursion vs Iteration**: Both approaches work well

## ⏱️ Complexity

- Time: O(n) - Visit each node twice at most
- Space: O(1) iterative, O(n/k) recursive (call stack)
