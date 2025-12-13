# 🎯 LeetCode #2699 - Modify Graph Edge Weights

## 📋 Today's Selection
| Item | Value |
|------|-------|
| 📂 Category | Graphs |
| 💻 Primary Language | Python |
| ⚡ Secondary Language | Java |
| 🔥 Difficulty | Hard |
| 🔗 Link | [LeetCode #2699](https://leetcode.com/problems/modify-graph-edge-weights/) |

---

## 📖 Problem Statement

You are given an **undirected weighted connected graph** containing `n` nodes labeled from `0` to `n - 1`, and an integer array `edges` where `edges[i] = [ai, bi, wi]` represents an edge between nodes `ai` and `bi` with weight `wi`.

Some edges have a weight of `-1` (**modifiable edges**), meaning you need to assign them **positive integer values** in the range `[1, 2 × 10^9]`.

Your task is to modify all edges with weight `-1` so that the **shortest distance** from `source` to `destination` is exactly equal to `target`.

**Return** the modified edges array. If multiple solutions exist, return any. If it's impossible, return an **empty array**.

### Constraints
- `1 <= n <= 100`
- `1 <= edges.length <= n * (n - 1) / 2`
- `edges[i].length == 3`
- `0 <= ai, bi < n`
- `wi = -1` or `1 <= wi <= 10^7`
- `ai != bi`
- `0 <= source, destination < n`
- `source != destination`
- `1 <= target <= 10^9`
- The graph is connected and has no self-loops or repeated edges.

---

## 💡 Approach Explanation

### 1. Intuition
This problem requires modifying edge weights to achieve an **exact shortest path distance**. The key insight is:
- Fixed edges cannot be changed
- Modifiable edges (`-1`) can be assigned values from `1` to `2×10^9`
- We need **exactly** the target distance, not less or more

### 2. Algorithm (Modified Dijkstra's)

#### Step 1: Initial Check with Fixed Edges Only
Run Dijkstra considering only fixed edges (treat `-1` edges as infinity).
- If shortest path `< target`: **Impossible** (can't increase fixed paths)

#### Step 2: Check with Minimum Modifiable Weights
Assign all `-1` edges weight `1`, run Dijkstra again.
- If shortest path `> target`: **Impossible** (even minimum weights too long)
- If shortest path `== target`: **Done!** Set unused `-1` edges to large value

#### Step 3: Binary Adjustment
For each `-1` edge on the path, adjust its weight to make the total exactly `target`.

### 3. Complexity
- **Time**: O(E² × log V) - Multiple Dijkstra runs
- **Space**: O(V + E) - Graph storage and priority queue

---

## ✅ Detailed Solution (Python)

```python
import heapq
from typing import List
from collections import defaultdict

class Solution:
    def modifiedGraphEdges(
        self, 
        n: int, 
        edges: List[List[int]], 
        source: int, 
        destination: int, 
        target: int
    ) -> List[List[int]]:
        
        INF = 2 * 10**9
        
        def dijkstra(ignore_modifiable: bool) -> int:
            """
            Run Dijkstra's algorithm from source to destination.
            If ignore_modifiable is True, treat -1 edges as infinity.
            """
            graph = defaultdict(list)
            
            for i, (u, v, w) in enumerate(edges):
                if w == -1:
                    if ignore_modifiable:
                        continue  # Skip modifiable edges
                    w = 1  # Use minimum weight for modifiable edges
                graph[u].append((v, w, i))
                graph[v].append((u, w, i))
            
            dist = [float('inf')] * n
            dist[source] = 0
            pq = [(0, source)]
            
            while pq:
                d, node = heapq.heappop(pq)
                
                if d > dist[node]:
                    continue
                    
                for neighbor, weight, _ in graph[node]:
                    new_dist = d + weight
                    if new_dist < dist[neighbor]:
                        dist[neighbor] = new_dist
                        heapq.heappush(pq, (new_dist, neighbor))
            
            return dist[destination]
        
        # Step 1: Check with only fixed edges
        dist_fixed = dijkstra(ignore_modifiable=True)
        if dist_fixed < target:
            return []  # Impossible - fixed path already shorter
        
        # Step 2: Check with minimum modifiable weights (all -1 -> 1)
        dist_min = dijkstra(ignore_modifiable=False)
        if dist_min > target:
            return []  # Impossible - even with min weights, too long
        
        # Step 3: Adjust modifiable edges to achieve exact target
        for i, (u, v, w) in enumerate(edges):
            if w != -1:
                continue  # Skip fixed edges
            
            # Try setting this edge to 1 first
            edges[i][2] = 1
            
            # Calculate shortest path with current assignments
            current_dist = dijkstra(ignore_modifiable=False)
            
            if current_dist <= target:
                # Adjust this edge to make distance exactly target
                edges[i][2] += target - current_dist
                
                # Set remaining -1 edges to INF (won't be on shortest path)
                for j in range(i + 1, len(edges)):
                    if edges[j][2] == -1:
                        edges[j][2] = INF
                
                return edges
        
        return edges
```

---

## ⚡ Concise Solution (Java)

```java
class Solution {
    private static final int INF = (int) 2e9;
    
    public int[][] modifiedGraphEdges(int n, int[][] edges, int source, int destination, int target) {
        long distFixed = dijkstra(n, edges, source, destination, true);
        if (distFixed < target) return new int[0][];
        
        long distMin = dijkstra(n, edges, source, destination, false);
        if (distMin > target) return new int[0][];
        
        for (int[] edge : edges) {
            if (edge[2] != -1) continue;
            edge[2] = 1;
            long curr = dijkstra(n, edges, source, destination, false);
            if (curr <= target) {
                edge[2] += target - curr;
                for (int[] e : edges) if (e[2] == -1) e[2] = INF;
                return edges;
            }
        }
        return edges;
    }
    
    private long dijkstra(int n, int[][] edges, int src, int dest, boolean ignoreModifiable) {
        List<int[]>[] graph = new ArrayList[n];
        for (int i = 0; i < n; i++) graph[i] = new ArrayList<>();
        
        for (int[] e : edges) {
            int w = e[2] == -1 ? (ignoreModifiable ? INF : 1) : e[2];
            graph[e[0]].add(new int[]{e[1], w});
            graph[e[1]].add(new int[]{e[0], w});
        }
        
        long[] dist = new long[n];
        Arrays.fill(dist, Long.MAX_VALUE);
        dist[src] = 0;
        PriorityQueue<long[]> pq = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
        pq.offer(new long[]{0, src});
        
        while (!pq.isEmpty()) {
            long[] curr = pq.poll();
            if (curr[0] > dist[(int) curr[1]]) continue;
            for (int[] next : graph[(int) curr[1]]) {
                long newDist = curr[0] + next[1];
                if (newDist < dist[next[0]]) {
                    dist[next[0]] = newDist;
                    pq.offer(new long[]{newDist, next[0]});
                }
            }
        }
        return dist[dest];
    }
}
```

---

## 🧪 Test Cases

```python
# Test Case 1: Basic case - modifiable edges can achieve target
n = 5
edges = [[4,1,-1],[2,0,-1],[0,3,-1],[4,3,-1]]
source, destination, target = 0, 1, 5
# Expected: Modified edges that make shortest path = 5

# Test Case 2: Impossible - fixed edges already shorter than target
n = 3
edges = [[0,1,2],[1,2,3]]
source, destination, target = 0, 2, 10
# Expected: [] (fixed path is 5, can't increase to 10)

# Test Case 3: Impossible - even minimum weights too long
n = 4
edges = [[0,1,-1],[1,2,-1],[2,3,-1]]
source, destination, target = 0, 3, 1
# Expected: [] (minimum path length is 3 with all edges = 1)

# Test Case 4: Already achieved with minimum weights
n = 3
edges = [[0,1,-1],[1,2,-1]]
source, destination, target = 0, 2, 2
# Expected: [[0,1,1],[1,2,1]] (minimum weights achieve target)

# Test Case 5: Mix of fixed and modifiable edges
n = 4
edges = [[0,1,5],[1,2,-1],[0,2,3],[2,3,-1]]
source, destination, target = 0, 3, 7
# Expected: Edges modified to achieve path length 7
```

---

## 🔗 Similar Problems

| # | Problem | Difficulty | Key Concept |
|---|---------|------------|-------------|
| 743 | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium | Dijkstra basics |
| 787 | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Medium | Modified Dijkstra |
| 1334 | [Find the City With Smallest Number of Neighbors](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) | Medium | Floyd-Warshall |
| 1514 | [Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) | Medium | Dijkstra variant |

---

## 📝 Key Takeaways

1. **Dijkstra's Algorithm** - Essential for weighted shortest path problems
2. **Binary Search on Answer** - Often useful when modifying edge weights
3. **Edge Cases** - Always check impossible scenarios first
4. **Greedy Assignment** - Assign minimum first, adjust incrementally

---

## 📊 Progress Tracker

| Date | Status | Time Spent | Notes |
|------|--------|------------|-------|
| 2025-12-13 | ⏳ In Progress | - | Hard graph problem requiring modified Dijkstra's |
