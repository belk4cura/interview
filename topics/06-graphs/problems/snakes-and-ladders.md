# Snakes and Ladders

## 🎯 What Are We Trying To Solve?

Finding the minimum number of dice rolls to win Snakes and Ladders is like finding the shortest path through a game board with teleportation points. Some squares (ladders) boost you forward, while others (snakes) send you backward. We need BFS to find the optimal path!

## 📝 The Problem

Given a Snakes and Ladders board (squares 1-100), find the minimum number of dice rolls needed to reach square 100 from square 1. You can roll 1-6 on each turn. Ladders move you up instantly, snakes move you down instantly.

```
Example:
Ladders: 32→62, 42→68, 12→98
Snakes: 95→13, 97→25, 93→37, 79→27, 75→19, 49→11, 47→17

Optimal path might be:
1 → 7 (roll 6) → 12 (roll 5) → 98 (ladder) → 100 (roll 2)
Minimum rolls: 3
```

## 💻 The Code

```javascript
/**
 * Find minimum dice rolls in Snakes and Ladders
 * @param {number[][]} ladders - Array of [start, end] ladder positions
 * @param {number[][]} snakes - Array of [start, end] snake positions
 * @returns {number} - Minimum rolls to reach 100, or -1 if impossible
 */
function snakesAndLadders(ladders, snakes) {
    // Create board with snakes and ladders
    const board = new Array(101).fill(0);
    
    // Add ladders (positive jumps)
    for (const [start, end] of ladders) {
        board[start] = end;
    }
    
    // Add snakes (negative jumps)
    for (const [start, end] of snakes) {
        board[start] = end;
    }
    
    // BFS to find shortest path
    const visited = new Array(101).fill(false);
    const queue = [{position: 1, rolls: 0}];
    visited[1] = true;
    
    while (queue.length > 0) {
        const {position, rolls} = queue.shift();
        
        // Check if we reached the goal
        if (position === 100) {
            return rolls;
        }
        
        // Try all possible dice rolls (1-6)
        for (let dice = 1; dice <= 6; dice++) {
            let nextPos = position + dice;
            
            // Can't go beyond square 100
            if (nextPos > 100) continue;
            
            // Check for snake or ladder at this position
            if (board[nextPos] !== 0) {
                nextPos = board[nextPos];
            }
            
            // Visit if not visited before
            if (!visited[nextPos]) {
                visited[nextPos] = true;
                queue.push({position: nextPos, rolls: rolls + 1});
            }
        }
    }
    
    return -1; // Can't reach square 100
}

/**
 * Alternative: Track the path taken
 */
function snakesAndLaddersWithPath(ladders, snakes) {
    const board = new Array(101).fill(0);
    
    for (const [start, end] of ladders) {
        board[start] = end;
    }
    
    for (const [start, end] of snakes) {
        board[start] = end;
    }
    
    const visited = new Map();
    const queue = [{position: 1, rolls: 0, path: [1]}];
    visited.set(1, 0);
    
    while (queue.length > 0) {
        const {position, rolls, path} = queue.shift();
        
        if (position === 100) {
            return {rolls, path};
        }
        
        for (let dice = 1; dice <= 6; dice++) {
            let nextPos = position + dice;
            
            if (nextPos > 100) continue;
            
            const actualNext = board[nextPos] || nextPos;
            
            if (!visited.has(actualNext) || visited.get(actualNext) > rolls + 1) {
                visited.set(actualNext, rolls + 1);
                queue.push({
                    position: actualNext,
                    rolls: rolls + 1,
                    path: [...path, actualNext]
                });
            }
        }
    }
    
    return {rolls: -1, path: []};
}

/**
 * LeetCode variant: 2D board with alternating direction
 */
function snakesAndLaddersMatrix(board) {
    const n = board.length;
    
    // Convert 2D board to 1D with proper numbering
    function getPosition(num) {
        const row = Math.floor((num - 1) / n);
        const col = (num - 1) % n;
        
        // Odd rows go right to left
        const actualCol = row % 2 === 0 ? col : n - 1 - col;
        const actualRow = n - 1 - row;
        
        return [actualRow, actualCol];
    }
    
    const visited = new Set([1]);
    const queue = [{square: 1, moves: 0}];
    
    while (queue.length > 0) {
        const {square, moves} = queue.shift();
        
        if (square === n * n) {
            return moves;
        }
        
        for (let dice = 1; dice <= 6; dice++) {
            let next = square + dice;
            
            if (next > n * n) break;
            
            const [row, col] = getPosition(next);
            
            // Check for snake or ladder
            if (board[row][col] !== -1) {
                next = board[row][col];
            }
            
            if (!visited.has(next)) {
                visited.add(next);
                queue.push({square: next, moves: moves + 1});
            }
        }
    }
    
    return -1;
}

/**
 * Optimize with bidirectional BFS
 */
function snakesAndLaddersBidirectional(ladders, snakes) {
    const board = new Array(101).fill(0);
    const reverseBoard = new Array(101).fill(null);
    
    for (const [start, end] of ladders) {
        board[start] = end;
        if (!reverseBoard[end]) reverseBoard[end] = [];
        reverseBoard[end].push(start);
    }
    
    for (const [start, end] of snakes) {
        board[start] = end;
        if (!reverseBoard[end]) reverseBoard[end] = [];
        reverseBoard[end].push(start);
    }
    
    // Forward BFS from 1
    const visitedForward = new Map([[1, 0]]);
    let queueForward = [1];
    
    // Backward BFS from 100
    const visitedBackward = new Map([[100, 0]]);
    let queueBackward = [100];
    
    let moves = 0;
    
    while (queueForward.length > 0 && queueBackward.length > 0) {
        // Expand smaller frontier
        if (queueForward.length <= queueBackward.length) {
            const nextQueue = [];
            moves++;
            
            for (const pos of queueForward) {
                for (let dice = 1; dice <= 6; dice++) {
                    let next = pos + dice;
                    if (next > 100) continue;
                    
                    const actual = board[next] || next;
                    
                    if (visitedBackward.has(actual)) {
                        return moves + visitedBackward.get(actual);
                    }
                    
                    if (!visitedForward.has(actual)) {
                        visitedForward.set(actual, moves);
                        nextQueue.push(actual);
                    }
                }
            }
            
            queueForward = nextQueue;
        } else {
            // Similar expansion from backward
            // (Implementation omitted for brevity)
        }
    }
    
    return -1;
}

// Test cases
const ladders1 = [[32, 62], [42, 68], [12, 98]];
const snakes1 = [[95, 13], [97, 25], [93, 37], [79, 27]];
console.log(snakesAndLadders(ladders1, snakes1)); // Minimum rolls

const ladders2 = [[2, 21], [4, 7], [10, 25], [19, 28]];
const snakes2 = [[26, 0], [20, 8], [16, 3], [18, 6]];
console.log(snakesAndLadders(ladders2, snakes2));
```

## 🎨 Step-by-Step Walkthrough

Let's trace through a simple board:

```
Ladders: 3→22, 5→8
Snakes: 11→2

Step 1: Start at square 1
Queue: [{pos: 1, rolls: 0}]

Step 2: Roll 1-6 from square 1
Possible: 2, 3→22(ladder), 4, 5→8(ladder), 6, 7
Queue: [{pos: 2, rolls: 1}, {pos: 22, rolls: 1}, 
        {pos: 4, rolls: 1}, {pos: 8, rolls: 1}, ...]

Step 3: Process square 22 (closest to goal)
Roll 1-6: Can reach 23-28
Continue BFS...

Eventually reach square 100 with minimum rolls
```

## 📊 Visual Understanding

```
Board Layout (simplified 1-30):

30 ← 29 ← 28 ← 27 ← 26    (Row 3, right to left)
21 → 22 → 23 → 24 → 25    (Row 2, left to right)
20 ← 19 ← 18 ← 17 ← 16    (Row 1, right to left)
11 → 12 → 13 → 14 → 15    (Row 1, left to right)
10 ← 9  ← 8  ← 7  ← 6     (Row 0, right to left)
1  → 2  → 3  → 4  → 5     (Row 0, left to right)

With Snakes (S) and Ladders (L):
- Square 3: L→22 (jump up)
- Square 11: S→2 (slide down)

BFS explores:
Level 0: [1]
Level 1: [2-7] (one roll)
Level 2: [8-13, special jumps] (two rolls)
...
```

## ⏱️ Complexity Analysis

- **Time Complexity:** O(N) where N = 100 (board size)
  - Each square visited at most once
  - Each square has at most 6 neighbors
  
- **Space Complexity:** O(N)
  - Visited array: O(100)
  - Queue: O(100) worst case

## 🚫 Common Mistakes

1. **Not handling board boundaries**
   ```javascript
   // Wrong: Can go past square 100
   let nextPos = position + dice;
   
   // Right: Cap at square 100
   if (nextPos > 100) continue;
   ```

2. **Visiting snake/ladder destinations multiple times**
   ```javascript
   // Wrong: Check visited before snake/ladder
   if (!visited[position + dice]) {
       nextPos = board[position + dice] || position + dice;
   }
   
   // Right: Check visited after determining final position
   nextPos = board[position + dice] || position + dice;
   if (!visited[nextPos]) {
   ```

3. **Not handling consecutive snakes/ladders**
   ```javascript
   // Wrong: Only one jump
   if (board[next]) next = board[next];
   
   // Right: In standard game, only one jump per landing
   // But some variants might chain jumps
   ```

## 🔄 Variations

```javascript
// Modified dice (e.g., 1-4 instead of 1-6)
function customDiceSnakesAndLadders(ladders, snakes, diceMax) {
    // Same logic but dice loop goes 1 to diceMax
    for (let dice = 1; dice <= diceMax; dice++) {
        // ...
    }
}

// Multiple players racing
function multiPlayerSnakesAndLadders(ladders, snakes, numPlayers) {
    const positions = new Array(numPlayers).fill(1);
    let currentPlayer = 0;
    const rolls = [];
    
    while (!positions.includes(100)) {
        // Simulate dice roll for current player
        const dice = Math.floor(Math.random() * 6) + 1;
        // Move player and check snakes/ladders
        // Track number of turns
        currentPlayer = (currentPlayer + 1) % numPlayers;
    }
    
    return rolls;
}

// Find all shortest paths
function allShortestPaths(ladders, snakes) {
    // Use BFS but track all paths that reach each square
    // with the minimum number of moves
    const paths = new Map();
    paths.set(1, [[1]]);
    
    // Modified BFS that keeps all optimal paths
    // ...
}
```

## 🔗 Related Problems

- **Minimum Knight Moves** - Chess piece movement with BFS
- **Jump Game IV** - Array jumping with BFS
- **Word Ladder** - Transform words with BFS
- **Shortest Path in Binary Matrix** - Grid navigation

## 💡 Key Takeaways

1. **Graph Modeling**: Board game as implicit graph
2. **BFS for Shortest Path**: Guarantees minimum moves
3. **State Representation**: Position + number of rolls
4. **Teleportation Handling**: Snakes and ladders as forced moves
5. **Boundary Conditions**: Can't exceed square 100

This classic problem demonstrates BFS on a game board with special rules!
