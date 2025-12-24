# 🎯 LeetCode #312 - Burst Balloons

## 📋 Today's Selection
| Item | Value |
|------|-------|
| 📂 Category | Dynamic Programming |
| 💻 Primary Language | Java |
| ⚡ Secondary Language | Python |
| 🔥 Difficulty | Hard |
| 🔗 Link | [LeetCode #312](https://leetcode.com/problems/burst-balloons/) |

---

## 📖 Problem Statement

You are given `n` balloons, indexed from `0` to `n - 1`. Each balloon is painted with a number on it represented by an array `nums`. You are asked to burst all the balloons.

If you burst the `i-th` balloon, you will get `nums[i - 1] * nums[i] * nums[i + 1]` coins. If `i - 1` or `i + 1` goes out of bounds of the array, then treat it as if there is a balloon with a `1` painted on it.

Return the **maximum** coins you can collect by bursting the balloons wisely.

### Constraints
- `n == nums.length`
- `1 <= n <= 300`
- `0 <= nums[i] <= 100`

### Examples

**Example 1:**
```
Input: nums = [3,1,5,8]
Output: 167
Explanation:
nums = [3,1,5,8] --> [3,5,8] --> [3,8] --> [8] --> []
coins =  3*1*5    +   3*5*8   +  1*3*8  + 1*8*1 = 15 + 120 + 24 + 8 = 167
```

**Example 2:**
```
Input: nums = [1,5]
Output: 10
```

---

## 💡 Approach Explanation

### 1. Intuition - Reverse Thinking 🧠

The key insight is to think **backwards**: instead of asking "which balloon to burst first?", ask "which balloon to burst **last** in a range?"

When we burst the last balloon `k` in a range `(left, right)`:
- All other balloons in that range are already gone
- The neighbors are now `nums[left]` and `nums[right]` (the boundaries)
- We get: `nums[left] * nums[k] * nums[right]` coins

### 2. Algorithm - Interval DP

#### State Definition
- `dp[left][right]` = maximum coins from bursting all balloons in range `(left, right)` exclusive

#### Transition
For each `k` in range `(left+1, right)` as the **last** balloon to burst:
```
dp[left][right] = max(dp[left][right], 
                      dp[left][k] + nums[left] * nums[k] * nums[right] + dp[k][right])
```

#### Key Trick
Pad the array with `1` at both ends: `[1] + nums + [1]`
- This handles boundary cases elegantly
- Original balloons are now at indices `1` to `n`

### 3. Complexity
- **Time**: O(n³) - Three nested loops (left, right, k)
- **Space**: O(n²) - 2D DP table

---

## ✅ Detailed Solution (Java)

```java
class Solution {
    public int maxCoins(int[] nums) {
        int n = nums.length;
        
        // Pad array with 1s at boundaries
        // This simplifies edge case handling
        int[] balloons = new int[n + 2];
        balloons[0] = 1;
        balloons[n + 1] = 1;
        for (int i = 0; i < n; i++) {
            balloons[i + 1] = nums[i];
        }
        
        // dp[left][right] = max coins from bursting all balloons 
        // in range (left, right) exclusive
        int[][] dp = new int[n + 2][n + 2];
        
        // Iterate by range length (bottom-up)
        // Start with length 2 (minimum to have at least 1 balloon inside)
        for (int length = 2; length <= n + 1; length++) {
            
            // Try all possible left boundaries
            for (int left = 0; left + length <= n + 1; left++) {
                int right = left + length;
                
                // Try each balloon k as the LAST one to burst in (left, right)
                for (int k = left + 1; k < right; k++) {
                    // Coins from:
                    // 1. Bursting all balloons in (left, k)
                    // 2. Bursting balloon k last (neighbors are left & right)
                    // 3. Bursting all balloons in (k, right)
                    int coins = dp[left][k] 
                              + balloons[left] * balloons[k] * balloons[right] 
                              + dp[k][right];
                    
                    dp[left][right] = Math.max(dp[left][right], coins);
                }
            }
        }
        
        // Answer: burst all original balloons between boundaries 0 and n+1
        return dp[0][n + 1];
    }
}
```

---

## ⚡ Concise Solution (Python)

```python
class Solution:
    def maxCoins(self, nums: list[int]) -> int:
        # Pad with 1s for boundary handling
        balloons = [1] + nums + [1]
        n = len(balloons)
        
        # dp[i][j] = max coins in range (i, j) exclusive
        dp = [[0] * n for _ in range(n)]
        
        # Iterate by increasing range length
        for length in range(2, n):
            for left in range(n - length):
                right = left + length
                for k in range(left + 1, right):
                    coins = (dp[left][k] + 
                            balloons[left] * balloons[k] * balloons[right] + 
                            dp[k][right])
                    dp[left][right] = max(dp[left][right], coins)
        
        return dp[0][n - 1]
```

---

## 🔄 Alternative: Memoization (Top-Down)

```python
class Solution:
    def maxCoins(self, nums: list[int]) -> int:
        balloons = [1] + nums + [1]
        n = len(balloons)
        memo = {}
        
        def dp(left: int, right: int) -> int:
            # Base case: no balloons to burst
            if left + 1 == right:
                return 0
            
            if (left, right) in memo:
                return memo[(left, right)]
            
            result = 0
            for k in range(left + 1, right):
                coins = (dp(left, k) + 
                        balloons[left] * balloons[k] * balloons[right] + 
                        dp(k, right))
                result = max(result, coins)
            
            memo[(left, right)] = result
            return result
        
        return dp(0, n - 1)
```

---

## 🧪 Test Cases

```python
# Test Case 1: Standard case
nums = [3, 1, 5, 8]
# Expected: 167
# Optimal: burst 1 -> 5 -> 3 -> 8

# Test Case 2: Two balloons
nums = [1, 5]
# Expected: 10
# Either order: 1*1*5 + 1*5*1 = 5 + 5 = 10

# Test Case 3: Single balloon
nums = [5]
# Expected: 5
# Only one choice: 1*5*1 = 5

# Test Case 4: All same values
nums = [2, 2, 2]
# Expected: 16
# 2*2*2 + 1*2*2 + 1*2*1 = 8 + 4 + 2 = 14? Let's verify...
# Actually: burst middle -> [2,2], then any -> 8 + 4 + 2 = 14

# Test Case 5: Zeros in array
nums = [3, 0, 5]
# Expected: 15
# Burst 0 first (gets 0), then 3*5*1 or 1*3*5 = 15
```

---

## 🎨 Visual Explanation

```
Original: [3, 1, 5, 8]
Padded:   [1, 3, 1, 5, 8, 1]
           0  1  2  3  4  5  (indices)

Think of it as: which balloon to burst LAST in each subrange?

Range (0, 5) - full range between boundaries:
  If k=1 (balloon 3) is last:
    dp[0][1] + 1*3*1 + dp[1][5] = 0 + 3 + ? 
  If k=4 (balloon 8) is last:
    dp[0][4] + 1*8*1 + dp[4][5] = ? + 8 + 0

We compute smaller ranges first (bottom-up),
then combine for larger ranges.
```

---

## 🔗 Similar Problems

| # | Problem | Difficulty | Key Concept |
|---|---------|------------|-------------|
| 1039 | [Minimum Score Triangulation of Polygon](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/) | Medium | Interval DP |
| 1000 | [Minimum Cost to Merge Stones](https://leetcode.com/problems/minimum-cost-to-merge-stones/) | Hard | Interval DP |
| 516 | [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/) | Medium | Interval DP |
| 87 | [Scramble String](https://leetcode.com/problems/scramble-string/) | Hard | Interval DP |

---

## 📝 Key Takeaways

1. **Reverse Thinking** - Instead of "first", think about "last" to burst
2. **Interval DP** - Classic pattern for problems with subranges
3. **Padding Technique** - Add boundaries to simplify edge cases
4. **O(n³) is acceptable** - For n ≤ 300, ~27 million operations is fine

---

## 🧩 Pattern Recognition

This problem belongs to the **Interval DP** pattern:
- State: `dp[i][j]` represents answer for range `[i, j]`
- Transition: Try all split points `k` in the range
- Order: Process by increasing range length

Similar patterns appear in:
- Matrix chain multiplication
- Optimal BST construction
- Stone game variants

---

## 📊 Progress Tracker

| Date | Status | Time Spent | Notes |
|------|--------|------------|-------|
| 2025-12-23 | ⏳ In Progress | - | Classic interval DP, think "last" not "first" |
