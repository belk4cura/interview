# Task Scheduler

## 🎯 What Are We Trying To Solve?

Imagine you're a CPU that needs to execute tasks, but there's a cooling period between identical tasks. It's like doing laundry - you can't wash the same type of clothes back-to-back, you need to wait for the machine to cool down. How do you schedule tasks to minimize total time?

## 📝 The Problem

Given an array of tasks represented by letters A-Z and a cooling period `n`, find the minimum time needed to execute all tasks. The same task must be separated by at least `n` intervals (the task can run again after `n` other tasks or idle periods).

```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8

Execution: A → B → idle → A → B → idle → A → B
Time units: 1    2    3     4    5    6     7    8
```

## 🧠 How to Think About This

The key insight: The most frequent task determines the minimum time! If task A appears most often, we need to ensure enough gaps between A's, and fill those gaps with other tasks.

### Visual Process
```
Tasks: [A,A,A,B,B,B], cooldown = 2

Most frequent: A and B (both appear 3 times)

Frame structure:
A → _ → _ → A → _ → _ → A
B → _ → _ → B → _ → _ → B

Interleave:
A → B → idle → A → B → idle → A → B
```

## 💻 The Code

### Approach 1: Math Formula (Optimal)

```javascript
/**
 * Calculate minimum time using mathematical approach
 * Time: O(n) where n is number of tasks
 * Space: O(1) - at most 26 different tasks
 */
function leastInterval(tasks, n) {
    // Count frequency of each task (A-Z = 26 possible)
    const freq = new Array(26).fill(0);
    for (const task of tasks) {
        freq[task.charCodeAt(0) - 65]++;
    }
    
    // Sort to find most frequent
    freq.sort((a, b) => b - a);
    
    // The most frequent task determines minimum time
    const maxFreq = freq[0];
    
    // Calculate idle slots needed
    // (maxFreq - 1): Number of gaps between most frequent task
    // * n: Cooldown period for each gap
    let idleSlots = (maxFreq - 1) * n;
    
    // Fill idle slots with other tasks
    for (let i = 1; i < 26 && freq[i] > 0; i++) {
        // Can place at most (maxFreq - 1) of each task in gaps
        // Why -1? The last execution doesn't need cooldown
        idleSlots -= Math.min(freq[i], maxFreq - 1);
    }
    
    // If negative, means all slots filled (no idle time needed)
    idleSlots = Math.max(0, idleSlots);
    
    // Total time = all tasks + remaining idle time
    return tasks.length + idleSlots;
}

// Test cases
console.log(leastInterval(["A","A","A","B","B","B"], 2)); // 8
console.log(leastInterval(["A","A","A","B","B","B"], 0)); // 6 (no cooldown)
console.log(leastInterval(["A","A","A","A","A","A","B","C","D","E","F","G"], 2)); // 16
```

### Approach 2: Priority Queue Simulation

```javascript
/**
 * Simulate task execution using priority queue
 * Time: O(n * m) where m is number of unique tasks
 * Space: O(m)
 */
function leastIntervalPQ(tasks, n) {
    // Count frequencies
    const freqMap = new Map();
    for (const task of tasks) {
        freqMap.set(task, (freqMap.get(task) || 0) + 1);
    }
    
    // Max heap of frequencies (using array for simplicity)
    const maxHeap = Array.from(freqMap.values()).sort((a, b) => b - a);
    
    let time = 0;
    
    while (maxHeap.length > 0) {
        const temp = [];
        let cycle = n + 1; // Process n+1 tasks per cycle
        
        // Try to execute different tasks in this cycle
        while (cycle > 0 && maxHeap.length > 0) {
            const freq = maxHeap.shift(); // Get most frequent
            
            if (freq > 1) {
                temp.push(freq - 1); // Still has executions left
            }
            
            time++;
            cycle--;
        }
        
        // Re-add tasks that still need execution
        temp.forEach(freq => maxHeap.push(freq));
        maxHeap.sort((a, b) => b - a); // Re-sort
        
        // Add idle time if cycle not complete and tasks remain
        if (maxHeap.length > 0) {
            time += cycle; // Add remaining idle slots
        }
    }
    
    return time;
}
```

## 🎨 Step-by-Step Walkthrough

Let's trace through `["A","A","A","B","B","B"], n = 2`:

```
Step 1: Count frequencies
A: 3, B: 3

Step 2: Calculate using formula
maxFreq = 3 (both A and B appear 3 times)

Step 3: Calculate idle slots
idleSlots = (3 - 1) * 2 = 4
Visualization:
A _ _ A _ _ A
  ↑ 2 gaps of size 2 = 4 slots

Step 4: Fill with other tasks
Task B appears 3 times
Can place min(3, 3-1) = 2 B's in gaps
idleSlots = 4 - 2 = 2

Visualization:
A B _ A B _ A B
    ↑     ↑ 2 idle slots remain

Step 5: Total time
tasks.length + idleSlots = 6 + 2 = 8

Final schedule: A B idle A B idle A B
```

## 📊 Why the Formula Works

### The Mathematical Insight

```
Most frequent task creates the "frame":
If A appears f times with cooldown n:

A [n slots] A [n slots] A [n slots] ... A
└────┬────┘ └────┬────┘ └────┬────┘
  Frame 1     Frame 2     Frame 3

Total frames: f-1 (last A doesn't need cooldown)
Slots per frame: n
Total slots: (f-1) * n

These slots can be filled by other tasks!
```

### Edge Cases

```
Case 1: Many different tasks
tasks = ["A","B","C","D","E","F"], n = 2
No task repeats, so no idle time needed
Time = 6

Case 2: One task dominates
tasks = ["A","A","A","A"], n = 3
A _ _ _ A _ _ _ A _ _ _ A
Time = 13

Case 3: Multiple tasks with max frequency
tasks = ["A","A","B","B","C","C"], n = 1
Can interleave perfectly: A B C A B C
Time = 6
```

## 🚫 Common Mistakes

1. **Not Handling Multiple Max-Frequency Tasks**
   ```javascript
   // Wrong: Only considering one max frequency task
   const slots = (maxFreq - 1) * n;
   
   // Right: Check how many tasks have max frequency
   let maxCount = 0;
   for (const f of freq) {
       if (f === maxFreq) maxCount++;
   }
   // Adjust formula accordingly
   ```

2. **Forgetting the Last Execution**
   ```javascript
   // Wrong: Using maxFreq instead of (maxFreq - 1)
   let idleSlots = maxFreq * n;
   
   // Right: Last execution doesn't need cooldown
   let idleSlots = (maxFreq - 1) * n;
   ```

3. **Not Capping Task Placement**
   ```javascript
   // Wrong: Placing all instances of a task
   idleSlots -= freq[i];
   
   // Right: Can only place (maxFreq - 1) in gaps
   idleSlots -= Math.min(freq[i], maxFreq - 1);
   ```

## 🔄 Alternative: Greedy with Cooling Queue

```javascript
function leastIntervalQueue(tasks, n) {
    const freq = new Map();
    for (const task of tasks) {
        freq.set(task, (freq.get(task) || 0) + 1);
    }
    
    // Sort tasks by frequency
    const sortedTasks = Array.from(freq.entries())
        .sort((a, b) => b[1] - a[1]);
    
    let time = 0;
    const cooldown = new Map();
    
    while (sortedTasks.some(([_, count]) => count > 0)) {
        // Find executable task with highest frequency
        let executed = false;
        
        for (const [task, count] of sortedTasks) {
            if (count > 0 && (!cooldown.has(task) || cooldown.get(task) <= time)) {
                // Execute this task
                time++;
                sortedTasks.find(t => t[0] === task)[1]--;
                cooldown.set(task, time + n);
                executed = true;
                break;
            }
        }
        
        // If no task could execute, idle
        if (!executed) {
            time++;
        }
        
        // Re-sort for next iteration
        sortedTasks.sort((a, b) => b[1] - a[1]);
    }
    
    return time;
}
```

## 🔗 Related Problems

- **Reorganize String** - Similar cooldown concept for characters
- **Rearrange String k Distance Apart** - Generalized version
- **Meeting Rooms II** - Interval scheduling
- **CPU Interval Scheduling** - Real CPU scheduling algorithms

## 💡 Key Takeaways

1. **Most Frequent Determines Time**: The bottleneck is the most common task
2. **Fill the Gaps**: Use other tasks to fill cooldown periods
3. **Math > Simulation**: Formula approach is O(n) vs O(n²) simulation
4. **Greedy Works**: Always schedule the most frequent available task

This problem models real CPU scheduling and is directly applicable to system design!
