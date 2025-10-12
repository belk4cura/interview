# Insert Node at Position

## 🎯 What Are We Trying To Solve?

Imagine you're managing a playlist where songs are numbered starting from 0. You want to insert a new song at a specific position - maybe your new favorite song should be the 3rd one in the playlist. This is exactly what inserting at a specific position does!

## 📝 The Problem

Given a linked list, a value to insert, and a position, insert a new node with that value at the specified position. The first element is at position 0.

Special cases:
- If the list is empty, the new node becomes the head
- If position is 0, insert at the head
- If position is beyond the list length, insert at the end

```
Example:
List: 1 → 2 → 4 → 5
Insert: 3 at position 2
Result: 1 → 2 → 3 → 4 → 5
```

## 📊 Examples

**Example 1:**
```
Input: head = [1, 2, 4], data = 3, position = 2
Output: [1, 2, 3, 4]

Visual:
Before: 1 → 2 → 4 → null
           ↑
    Position 0,1,2
After:  1 → 2 → 3 → 4 → null
```

**Example 2:**
```
Input: head = [1, 2, 3], data = 0, position = 0
Output: [0, 1, 2, 3]

Visual:
Before: 1 → 2 → 3 → null
After:  0 → 1 → 2 → 3 → null
```

**Example 3:**
```
Input: head = null, data = 5, position = 0
Output: [5]

Visual:
Before: null (empty list)
After:  5 → null
```

## 💻 The Code

```javascript
// Definition for singly-linked list node
class ListNode {
    constructor(data = 0, next = null) {
        this.data = data;
        this.next = next;
    }
}

/**
 * Insert a node at a specific position in a linked list
 * @param {ListNode} head - Head of the linked list
 * @param {number} data - Data to insert
 * @param {number} position - Position to insert at (0-indexed)
 * @returns {ListNode} - Head of the modified list
 */
function insertNodeAtPosition(head, data, position) {
    // Create the new node to insert
    const newNode = new ListNode(data);
    
    // Case 1: Empty list or insert at head (position 0)
    if (head === null || position === 0) {
        newNode.next = head;
        return newNode;
    }
    
    // Case 2: Insert at any other position
    let current = head;
    let index = 0;
    
    // Traverse to the node BEFORE the insertion point
    // Stop if we reach position-1 or the end of the list
    while (current.next !== null && index < position - 1) {
        current = current.next;
        index++;
    }
    
    // Insert the new node
    // Save the rest of the list
    const temp = current.next;
    // Point current to new node
    current.next = newNode;
    // Point new node to the rest of the list
    newNode.next = temp;
    
    return head;
}

// Alternative: More explicit with helper function
function insertNodeAtPositionWithHelper(head, data, position) {
    const newNode = new ListNode(data);
    
    if (head === null) {
        return newNode;
    }
    
    if (position === 0) {
        newNode.next = head;
        return newNode;
    }
    
    let current = head;
    let index = 0;
    
    // Find the node before insertion point
    while (current.next !== null && index < position - 1) {
        current = current.next;
        index++;
    }
    
    // Use helper to insert between nodes
    insertBetween(current, newNode);
    return head;
}

function insertBetween(current, newNode) {
    const temp = current.next;
    current.next = newNode;
    newNode.next = temp;
}

// Alternative: With bounds checking
function insertNodeAtPositionSafe(head, data, position) {
    if (position < 0) {
        throw new Error("Position cannot be negative");
    }
    
    const newNode = new ListNode(data);
    
    if (position === 0 || head === null) {
        newNode.next = head;
        return newNode;
    }
    
    let current = head;
    let index = 0;
    
    // Traverse to position-1
    while (index < position - 1) {
        if (current.next === null) {
            // Position is beyond list length, insert at end
            current.next = newNode;
            return head;
        }
        current = current.next;
        index++;
    }
    
    newNode.next = current.next;
    current.next = newNode;
    return head;
}

// Helper functions for testing
function arrayToLinkedList(arr) {
    if (!arr || arr.length === 0) return null;
    
    const head = new ListNode(arr[0]);
    let current = head;
    
    for (let i = 1; i < arr.length; i++) {
        current.next = new ListNode(arr[i]);
        current = current.next;
    }
    
    return head;
}

function linkedListToArray(head) {
    const result = [];
    let current = head;
    
    while (current !== null) {
        result.push(current.data);
        current = current.next;
    }
    
    return result;
}

// Test cases
let list1 = arrayToLinkedList([1, 2, 4]);
list1 = insertNodeAtPosition(list1, 3, 2);
console.log(linkedListToArray(list1));  // [1, 2, 3, 4]

let list2 = arrayToLinkedList([1, 2, 3]);
list2 = insertNodeAtPosition(list2, 0, 0);
console.log(linkedListToArray(list2));  // [0, 1, 2, 3]

let list3 = null;
list3 = insertNodeAtPosition(list3, 5, 0);
console.log(linkedListToArray(list3));  // [5]

let list4 = arrayToLinkedList([1, 2]);
list4 = insertNodeAtPosition(list4, 3, 5);  // Beyond length
console.log(linkedListToArray(list4));  // [1, 2, 3]
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `insertNodeAtPosition([1, 2, 4], 3, 2)`:

```
Initial state:
head → 1 → 2 → 4 → null
Position:  0   1   2

Target: Insert 3 at position 2

Step 1: Create new node
newNode = ListNode(3)

Step 2: Check special cases
- head !== null ✓
- position !== 0 ✓
Continue to normal insertion

Step 3: Traverse to position-1 (position 1)
current = head (position 0)
index = 0
0 < 2-1? Yes, move forward

current = current.next (position 1)
index = 1
1 < 2-1? No, stop here

Step 4: Insert the new node
temp = current.next (node with 4)
current.next = newNode (2 now points to 3)
newNode.next = temp (3 now points to 4)

Result: 1 → 2 → 3 → 4 → null
```

## 📊 Visual Understanding

```
Inserting at Position 2:

Step 1: Find position-1 (index 1)
[1] → [2] → [4] → null
       ↑
    current
    (pos 1)

Step 2: Create new node
[3] → null
 ↑
newNode

Step 3: Rewire connections
[1] → [2]   [4] → null
       ↓     ↑
      [3] ──┘

Final: [1] → [2] → [3] → [4] → null
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(n)
  - Worst case: traverse to the end of the list
  - n is the number of nodes in the list

- **Space Complexity:** O(1)
  - Only create one new node
  - Use a constant amount of extra variables

## 🚫 Common Mistakes

1. **Off-by-one error in positioning**
   ```javascript
   // Wrong: Going to position instead of position-1
   while (index < position) {  // Goes too far!
       current = current.next;
       index++;
   }
   
   // Right: Stop at position-1
   while (index < position - 1) {
       current = current.next;
       index++;
   }
   ```

2. **Not handling position 0 correctly**
   ```javascript
   // Wrong: Trying to traverse when position is 0
   let current = head;
   while (index < position - 1) {  // -1 when position is 0!
   
   // Right: Handle position 0 specially
   if (position === 0) {
       newNode.next = head;
       return newNode;
   }
   ```

3. **Losing the rest of the list**
   ```javascript
   // Wrong: Not saving the rest
   current.next = newNode;
   newNode.next = null;  // Lost everything after!
   
   // Right: Save and reconnect
   const temp = current.next;
   current.next = newNode;
   newNode.next = temp;
   ```

## 🔄 Variations

```javascript
// Insert before a specific value
function insertBeforeValue(head, data, targetValue) {
    const newNode = new ListNode(data);
    
    if (head === null || head.data === targetValue) {
        newNode.next = head;
        return newNode;
    }
    
    let current = head;
    while (current.next !== null && current.next.data !== targetValue) {
        current = current.next;
    }
    
    newNode.next = current.next;
    current.next = newNode;
    return head;
}

// Insert after a specific value
function insertAfterValue(head, data, targetValue) {
    if (head === null) return null;
    
    let current = head;
    while (current !== null && current.data !== targetValue) {
        current = current.next;
    }
    
    if (current !== null) {
        const newNode = new ListNode(data);
        newNode.next = current.next;
        current.next = newNode;
    }
    
    return head;
}

// Insert sorted (maintain sorted order)
function insertSorted(head, data) {
    const newNode = new ListNode(data);
    
    if (head === null || head.data >= data) {
        newNode.next = head;
        return newNode;
    }
    
    let current = head;
    while (current.next !== null && current.next.data < data) {
        current = current.next;
    }
    
    newNode.next = current.next;
    current.next = newNode;
    return head;
}
```

## 🔗 Related Problems

- **Insert at Head** - Special case of position 0
- **Insert at Tail** - Special case of last position
- **Delete Node at Position** - Opposite operation
- **Get Node at Position** - Access without modification

## 💡 Key Takeaways

1. **Position-1 Pattern**: To insert at position n, stop at position n-1
2. **Edge Cases**: Handle empty list and position 0 specially
3. **Preserve Connections**: Always save existing connections before rewiring
4. **0-Indexing**: First element is at position 0, not 1

This is a fundamental operation that combines traversal with pointer manipulation!
