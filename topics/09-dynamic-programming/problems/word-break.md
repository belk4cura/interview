# Word Break

## 🎯 What Are We Trying To Solve?

Word break is like checking if a string can be segmented into dictionary words - like splitting "leetcode" into "leet" and "code" from a valid word list.

## 📝 The Problem

Given a string and a dictionary of words, determine if the string can be segmented into space-separated dictionary words.

```
Example:
s = "leetcode", wordDict = ["leet","code"]
Output: true

s = "applepenapple", wordDict = ["apple","pen"]
Output: true

s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
Output: false
```

## 💻 The Code

```javascript
/**
 * Dynamic Programming Solution
 * Time: O(n² × m), Space: O(n)
 */
function wordBreak(s, wordDict) {
    const wordSet = new Set(wordDict);
    const dp = new Array(s.length + 1).fill(false);
    dp[0] = true; // Empty string
    
    for (let i = 1; i <= s.length; i++) {
        for (let j = 0; j < i; j++) {
            if (dp[j] && wordSet.has(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }
    
    return dp[s.length];
}

/**
 * Word Break II - Return all segmentations
 * Time: O(2^n), Space: O(2^n)
 */
function wordBreakII(s, wordDict) {
    const wordSet = new Set(wordDict);
    const memo = new Map();
    
    function backtrack(start) {
        if (memo.has(start)) {
            return memo.get(start);
        }
        
        const result = [];
        
        if (start === s.length) {
            result.push("");
            return result;
        }
        
        for (let end = start + 1; end <= s.length; end++) {
            const word = s.substring(start, end);
            
            if (wordSet.has(word)) {
                const suffixes = backtrack(end);
                
                for (const suffix of suffixes) {
                    result.push(word + (suffix ? " " + suffix : ""));
                }
            }
        }
        
        memo.set(start, result);
        return result;
    }
    
    return backtrack(0);
}

/**
 * BFS Solution
 */
function wordBreakBFS(s, wordDict) {
    const wordSet = new Set(wordDict);
    const queue = [0];
    const visited = new Set();
    
    while (queue.length > 0) {
        const start = queue.shift();
        
        if (visited.has(start)) continue;
        visited.add(start);
        
        for (let end = start + 1; end <= s.length; end++) {
            if (wordSet.has(s.substring(start, end))) {
                if (end === s.length) return true;
                queue.push(end);
            }
        }
    }
    
    return false;
}
```

## 🎨 Key Insights

- **DP State**: dp[i] = can segment s[0...i-1]
- **Substring Check**: Try all possible word endings
- **Memoization**: Cache results for overlapping subproblems

## ⏱️ Complexity

- DP: O(n² × m) time where m = avg word length
- Space: O(n) for DP array
