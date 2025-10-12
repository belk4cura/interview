# Merge Two Sorted Lists

## 🎯 What Are We Trying To Solve?

Imagine you have two lines of people, both already sorted by height (shortest to tallest). You want to merge them into one line, keeping everyone in height order. You can only look at the front person of each line at a time.

This is the Merge Two Sorted Lists problem: combine two already-sorted linked lists into one sorted linked list.

## 📝 Examples

**Example 1:**
```
List 1: 1 → 2 → 4
List 2: 1 → 3 → 4
Output: 1 → 1 → 2 → 3 → 4 → 4
```

**Example 2:**
```
List 1: (empty)
List 2: 0
Output: 0
```

**Example 3:**
```
List 1: 5 → 10 → 15
List 2: 2 → 6 → 20
Output: 2 → 5 → 6 → 10 → 15 → 20
```

## 🧠 How Do We Think About This?

Think of it like merging two sorted decks of cards:
1. Look at the top card of each deck
2. Take the smaller card and put it in your result pile
3. Repeat until both decks are empty

The key insight: Since both lists are already sorted, we only need to compare the front elements!

## 💻 The Code

```javascript
// First, let's define what a node in our linked list looks like
class ListNode {
    constructor(value = 0, next = null) {
        this.value = value;  // The data this node holds
        this.next = next;    // Pointer to the next node
    }
}

function mergeTwoLists(list1, list2) {
    // Create a dummy node to simplify our code
    // This is like a placeholder at the start
    const dummy = new ListNode(0);
    
    // This pointer will help us build the new list
    let current = dummy;
    
    // While both lists have nodes remaining
    while (list1 && list2) {
        // Compare the values at the front of each list
        if (list1.value <= list2.value) {
            // list1's front is smaller or equal, use it
            current.next = list1;
            list1 = list1.next;  // Move list1 forward
        } else {
            // list2's front is smaller, use it
            current.next = list2;
            list2 = list2.next;  // Move list2 forward
        }
        // Move our building pointer forward
        current = current.next;
    }
    
    // One list might still have nodes left
    // Attach all remaining nodes (they're already sorted!)
    current.next = list1 || list2;
    
    // Return the real start (skip the dummy node)
    return dummy.next;
}

// Test the function
function createList(arr) {
    const dummy = new ListNode(0);
    let current = dummy;
    for (const val of arr) {
        current.next = new ListNode(val);
        current = current.next;
    }
    return dummy.next;
}

function printList(head) {
    const values = [];
    while (head) {
        values.push(head.value);
        head = head.next;
    }
    return values.join(' → ');
}

// Example usage
const list1 = createList([1, 2, 4]);
const list2 = createList([1, 3, 4]);
const merged = mergeTwoLists(list1, list2);
console.log(printList(merged)); // "1 → 1 → 2 → 3 → 4 → 4"
```

## 🎨 Step-by-Step Walkthrough

Let's trace through merging `[1, 2, 4]` and `[1, 3, 4]`:

```
Initial:
  list1: 1 → 2 → 4
  list2: 1 → 3 → 4
  dummy: 0 → (nothing yet)

Step 1: Compare 1 and 1
  - Both equal, take from list1
  - dummy: 0 → 1
  - list1: 2 → 4
  - list2: 1 → 3 → 4

Step 2: Compare 2 and 1
  - 1 is smaller, take from list2
  - dummy: 0 → 1 → 1
  - list1: 2 → 4
  - list2: 3 → 4

Step 3: Compare 2 and 3
  - 2 is smaller, take from list1
  - dummy: 0 → 1 → 1 → 2
  - list1: 4
  - list2: 3 → 4

Step 4: Compare 4 and 3
  - 3 is smaller, take from list2
  - dummy: 0 → 1 → 1 → 2 → 3
  - list1: 4
  - list2: 4

Step 5: Compare 4 and 4
  - Both equal, take from list1
  - dummy: 0 → 1 → 1 → 2 → 3 → 4
  - list1: (empty)
  - list2: 4

Step 6: list1 is empty
  - Attach remaining list2
  - dummy: 0 → 1 → 1 → 2 → 3 → 4 → 4

Final: Return dummy.next = 1 → 1 → 2 → 3 → 4 → 4
```

## ⏱️ How Fast Is This?

- **Time Complexity:** O(n + m) where n and m are the lengths of the two lists
  - In plain English: If you have 100 nodes in list1 and 50 in list2, you'll do about 150 operations
  
- **Space Complexity:** O(1) - We're reusing existing nodes!
  - In plain English: We don't create new nodes, just rearrange pointers

## 🚫 Common Mistakes

1. **Forgetting to handle empty lists**
   ```javascript
   // Wrong: Doesn't check if lists exist
   if (list1.value < list2.value) // Crashes if list1 is null!
   
   // Right: Check lists exist first
   while (list1 && list2) {
   ```

2. **Creating new nodes instead of reusing**
   ```javascript
   // Wrong: Creates unnecessary new nodes
   current.next = new ListNode(list1.value);
   
   // Right: Reuse existing nodes
   current.next = list1;
   ```

3. **Forgetting remaining nodes**
   ```javascript
   // After the while loop, one list might have nodes left
   // Don't forget: current.next = list1 || list2;
   ```

4. **Returning the dummy node**
   ```javascript
   // Wrong: Returns dummy (which has value 0)
   return dummy;
   
   // Right: Returns the real start
   return dummy.next;
   ```

## 🔄 Alternative Solutions

### Recursive Approach (Elegant but uses more memory)
```javascript
function mergeTwoListsRecursive(list1, list2) {
    // Base cases: if one list is empty, return the other
    if (!list1) return list2;
    if (!list2) return list1;
    
    // Compare and recursively merge
    if (list1.value <= list2.value) {
        list1.next = mergeTwoListsRecursive(list1.next, list2);
        return list1;
    } else {
        list2.next = mergeTwoListsRecursive(list1, list2.next);
        return list2;
    }
}
```
- Time: Still O(n + m)
- Space: O(n + m) due to recursion stack

### Without Dummy Node (More complex)
```javascript
function mergeTwoListsNoDummy(list1, list2) {
    // Handle empty lists
    if (!list1) return list2;
    if (!list2) return list1;
    
    // Determine the head
    let head;
    if (list1.value <= list2.value) {
        head = list1;
        list1 = list1.next;
    } else {
        head = list2;
        list2 = list2.next;
    }
    
    // Continue merging
    let current = head;
    while (list1 && list2) {
        if (list1.value <= list2.value) {
            current.next = list1;
            list1 = list1.next;
        } else {
            current.next = list2;
            list2 = list2.next;
        }
        current = current.next;
    }
    
    current.next = list1 || list2;
    return head;
}
```

## 🔗 Related Problems

- **Merge K Sorted Lists**: Merge multiple sorted lists
- **Sort List**: Sort an unsorted linked list
- **Remove Duplicates from Sorted List**: Clean up a sorted list
- **Merge Sorted Array**: Similar concept but with arrays

## 💡 Key Takeaway

The dummy node technique simplifies linked list problems by eliminating special cases for the head. This "merge" pattern is fundamental and appears in merge sort, database joins, and many other algorithms!
