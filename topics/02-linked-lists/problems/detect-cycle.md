# Detect Cycle in Linked List

## 🎯 What Are We Trying To Solve?

Imagine a circular racetrack where runners keep going around and around - they never reach an "end". This is what a cycle in a linked list looks like: at some point, a node points back to an earlier node, creating an infinite loop. We need to detect if such a cycle exists!

## 📝 The Problem

Given the head of a linked list, determine if the list has a cycle in it. A cycle exists if there is some node in the list that can be reached again by continuously following the `next` pointer.

```
Example of a cycle:
    1 → 2 → 3 → 4
        ↑       ↓
        └───────┘
        
No cycle:
    1 → 2 → 3 → 4 → null
```

## 📊 Examples

**Example 1:**
```
Input: head = [3,2,0,-4], pos = 1
(pos indicates where tail connects, -1 means no cycle)

Visual:
    3 → 2 → 0 → -4
        ↑        ↓
        └────────┘

Output: true
```

**Example 2:**
```
Input: head = [1,2], pos = 0

Visual:
    1 → 2
    ↑   ↓
    └───┘

Output: true
```

**Example 3:**
```
Input: head = [1], pos = -1

Visual:
    1 → null

Output: false
```

## 💻 The Code

```javascript
// Definition for singly-linked list node
class ListNode {
    constructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}

/**
 * Method 1: Using a Set to track visited nodes
 * Time: O(n), Space: O(n)
 * @param {ListNode} head
 * @returns {boolean}
 */
function hasCycleSet(head) {
    // Empty list has no cycle
    if (head === null) {
        return false;
    }
    
    // Keep track of nodes we've seen
    const visited = new Set();
    
    let current = head;
    while (current !== null) {
        // If we've seen this node before, there's a cycle
        if (visited.has(current)) {
            return true;
        }
        
        // Mark this node as visited
        visited.add(current);
        current = current.next;
    }
    
    // Reached the end without finding a cycle
    return false;
}

/**
 * Method 2: Floyd's Cycle Detection (Tortoise and Hare)
 * Time: O(n), Space: O(1) - More space efficient!
 * @param {ListNode} head
 * @returns {boolean}
 */
function hasCycleTwoPointers(head) {
    // Empty list or single node without self-loop
    if (head === null || head.next === null) {
        return false;
    }
    
    // Initialize two pointers
    let slow = head;        // Tortoise moves 1 step
    let fast = head.next;   // Hare moves 2 steps
    
    // Continue until fast reaches the end or meets slow
    while (fast !== null && fast.next !== null) {
        // If pointers meet, there's a cycle
        if (slow === fast) {
            return true;
        }
        
        // Move pointers at different speeds
        slow = slow.next;       // Move 1 step
        fast = fast.next.next;  // Move 2 steps
    }
    
    // Fast reached the end, no cycle
    return false;
}

/**
 * Alternative Floyd's implementation (both start at head)
 */
function hasCycleTwoPointersAlt(head) {
    if (head === null) {
        return false;
    }
    
    let slow = head;
    let fast = head;
    
    // Important: Check fast and fast.next before moving
    while (fast !== null && fast.next !== null) {
        slow = slow.next;
        fast = fast.next.next;
        
        // Check after moving
        if (slow === fast) {
            return true;
        }
    }
    
    return false;
}

/**
 * Extension: Find the start of the cycle
 * Returns the node where the cycle begins, or null if no cycle
 */
function detectCycleStart(head) {
    if (head === null || head.next === null) {
        return null;
    }
    
    // Phase 1: Detect if cycle exists
    let slow = head;
    let fast = head;
    let hasCycle = false;
    
    while (fast !== null && fast.next !== null) {
        slow = slow.next;
        fast = fast.next.next;
        
        if (slow === fast) {
            hasCycle = true;
            break;
        }
    }
    
    if (!hasCycle) {
        return null;
    }
    
    // Phase 2: Find the start of the cycle
    // Move slow back to head, keep fast at meeting point
    slow = head;
    
    // Move both one step at a time
    while (slow !== fast) {
        slow = slow.next;
        fast = fast.next;
    }
    
    return slow;  // This is the start of the cycle
}

// Helper function to create a cycle for testing
function createCycleList(values, pos) {
    if (!values || values.length === 0) return null;
    
    const head = new ListNode(values[0]);
    let current = head;
    let cycleNode = pos === 0 ? head : null;
    
    for (let i = 1; i < values.length; i++) {
        current.next = new ListNode(values[i]);
        current = current.next;
        if (i === pos) {
            cycleNode = current;
        }
    }
    
    // Create the cycle
    if (pos >= 0) {
        current.next = cycleNode;
    }
    
    return head;
}

// Test cases
const list1 = createCycleList([3, 2, 0, -4], 1);
console.log(hasCycleTwoPointers(list1));  // true

const list2 = createCycleList([1, 2], 0);
console.log(hasCycleTwoPointers(list2));  // true

const list3 = createCycleList([1], -1);
console.log(hasCycleTwoPointers(list3));  // false
```

## 🎨 Step-by-Step Walkthrough

### Method 1: Using a Set

```
List: 1 → 2 → 3 → 4
          ↑       ↓
          └───────┘

Step 1: current = 1, visited = {}
- Not in visited, add 1
- visited = {1}

Step 2: current = 2, visited = {1}
- Not in visited, add 2
- visited = {1, 2}

Step 3: current = 3, visited = {1, 2}
- Not in visited, add 3
- visited = {1, 2, 3}

Step 4: current = 4, visited = {1, 2, 3}
- Not in visited, add 4
- visited = {1, 2, 3, 4}

Step 5: current = 2, visited = {1, 2, 3, 4}
- 2 IS in visited!
- Return true (cycle detected)
```

### Method 2: Floyd's Algorithm (Tortoise and Hare)

```
Initial: 1 → 2 → 3 → 4
             ↑       ↓
             └───────┘
         
Step 1: slow = 1, fast = 2
Step 2: slow = 2, fast = 4
Step 3: slow = 3, fast = 3
- They meet! Cycle detected

Why does this work?
- If there's a cycle, fast will eventually "lap" slow
- Fast moves 2x speed, so it closes the gap by 1 each step
- They MUST meet if there's a cycle
```

## 📊 Visual Understanding

### Why Floyd's Algorithm Works

```
No Cycle:
slow →  [1] → [2] → [3] → [4] → null
fast →→ 

Fast reaches null → No cycle

With Cycle:
Distance between pointers decreases by 1 each step

Step 0: Gap = 3
    S
[1] → [2] → [3] → [4]
            F ↑     ↓
              └─────┘

Step 1: Gap = 2
      S
[1] → [2] → [3] → [4]
      ↑ F         ↓
      └───────────┘

Eventually: Gap = 0 (they meet)
```

## ⏱️ Complexity Analysis

**Method 1 (HashSet):**
- Time: O(n) - visit each node at most once
- Space: O(n) - store all nodes in set

**Method 2 (Two Pointers):**
- Time: O(n) - in worst case, fast pointer travels around cycle
- Space: O(1) - only two pointers, no extra storage!

## 🚫 Common Mistakes

1. **Not checking for null pointers**
   ```javascript
   // Wrong: Will crash on empty list or when reaching end
   while (fast.next !== null) {
       fast = fast.next.next;  // fast could be null!
   }
   
   // Right: Check both fast and fast.next
   while (fast !== null && fast.next !== null) {
       fast = fast.next.next;
   }
   ```

2. **Comparing values instead of nodes**
   ```javascript
   // Wrong: Different nodes might have same value
   if (slow.val === fast.val) {
       return true;
   }
   
   // Right: Compare node references
   if (slow === fast) {
       return true;
   }
   ```

3. **Starting pointers incorrectly**
   ```javascript
   // Be consistent with starting positions
   // Either both at head, or slow at head and fast at head.next
   // Just be careful with the loop conditions!
   ```

## 🔄 Variations

```javascript
// Find cycle length
function cycleLength(head) {
    if (!hasCycleTwoPointers(head)) {
        return 0;
    }
    
    // Find meeting point
    let slow = head, fast = head;
    while (fast && fast.next) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow === fast) break;
    }
    
    // Count nodes in cycle
    let count = 1;
    fast = fast.next;
    while (fast !== slow) {
        fast = fast.next;
        count++;
    }
    
    return count;
}

// Remove cycle
function removeCycle(head) {
    const start = detectCycleStart(head);
    if (start === null) return head;
    
    // Find the node just before cycle start
    let current = start;
    while (current.next !== start) {
        current = current.next;
    }
    
    // Break the cycle
    current.next = null;
    return head;
}
```

## 🔗 Related Problems

- **Find Cycle Start** - Locate where the cycle begins
- **Happy Number** - Cycle detection in number sequences
- **Find Duplicate Number** - Array cycle detection
- **Intersection of Two Linked Lists** - Similar two-pointer technique

## 💡 Key Takeaways

1. **Two Approaches**: Trade space for simplicity (Set) or use clever two-pointer technique
2. **Floyd's Algorithm**: Beautiful example of mathematical insight in programming
3. **Reference vs Value**: Always compare node references, not values
4. **Null Checks**: Critical for avoiding runtime errors

This is a classic interview problem that tests understanding of pointers and cycle detection!
