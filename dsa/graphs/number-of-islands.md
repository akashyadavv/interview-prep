# Number of Islands

`graphs / flood-fill` · **Difficulty:** Medium · **Source:** LC 200 · **Date:** 2026-07-20 · **Companies:** Amazon, Google

**Link:** https://leetcode.com/problems/number-of-islands/

---

## Rating — 3/5

| # | Signal | Score | Note |
|---|--------|:-----:|------|
| 1 | Explained approach out loud *before* coding | 2/5 | Posted code with "what's wrong" — no verbal plan, though the DFS flood-fill instinct was right |
| 2 | Reached optimal (or justified the tradeoff) | 5/5 | DFS flood fill is the optimal approach — O(m·n) |
| 3 | Code was clean & bug-free on first pass | 2/5 | Right skeleton, but four bugs — two of them crash the program |
| 4 | Complexity analysis correct & fast | 2/5 | Not stated in the session |
| 5 | **Verified with a hand-traced adversarial input** | 1/5 | No trace — a single non-square grid would have caught the headline bug instantly |

---

## My initial approach
> DFS flood fill: scan every cell, and when I hit an unvisited `'1'`, recursively mark the whole connected component as visited and bump the island count. Correct mental model.

## Where it broke
Four bugs, in order of severity:
1. **Flipped `visited` dimensions** — `[[False]*m for _ in range(n)]` builds an `n×m` grid instead of `m×n`. Crashes (`IndexError`) on any non-square grid.
2. **`range(...)` for directions** — `range` takes ints, not lists. Needs to iterate a list of tuples: `[(0,1),(0,-1),(1,0),(-1,0)]`.
3. **No bounds check + bare recursive call** — recursing into `i+row, j+col` without a bounds guard crashes at the grid edge; and `markIsland(...)` must be `self.markIsland(...)`.
4. **Redundant re-check** — the second `if grid == '1' and not visited` is always true after the guard clause. Dead code.

## Corrected solution
```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        m, n = len(grid), len(grid[0])
        count = 0
        visited = [[False] * n for _ in range(m)]      # m rows, n cols
        for i in range(m):
            for j in range(n):
                if grid[i][j] == '1' and not visited[i][j]:
                    self.markIsland(grid, visited, m, n, i, j)
                    count += 1
        return count

    def markIsland(self, grid, visited, m, n, i, j):
        if visited[i][j] or grid[i][j] == '0':
            return
        visited[i][j] = True
        for row, col in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            ni, nj = i + row, j + col
            if 0 <= ni < m and 0 <= nj < n:            # bounds before recursing
                self.markIsland(grid, visited, m, n, ni, nj)
```

## Complexity
- **Time:** O(m·n) — every cell is visited once; once marked, never re-entered
- **Space:** O(m·n) — the `visited` matrix, plus recursion stack up to O(m·n) worst case (grid entirely land → one deep DFS). Can drop the matrix by mutating `grid` in place (`'1'→'0'`), leaving only the stack.

## Verification trace
> Adversarial input: a **single-row, non-square grid** → `grid = [["1","1","0","1"]]`, so `m=1, n=4`.
>
> - **Original code:** `visited = [[False]*1 for _ in range(4)]` → 4 rows × 1 col. First access `visited[0][1]` → **IndexError**. Bug #1 exposed on the very first non-square input.
> - **Fixed code:** `visited` is 1×4. Scan → island at (0,0)-(0,1), then (0,3) → **count = 2** ✅

## Retro
- ✅ **Went well:** correct algorithm on sight — DFS flood fill with a visited set; skeleton was structurally sound
- ❌ **Went wrong:** three of four bugs (dimensions, bounds, `self.`) are exactly what one hand-trace catches; shipped without running or tracing anything
- 🔑 **Pattern trigger** — "count regions / connected components in a grid" → DFS *or* BFS flood fill; always build `visited` as `[[False]*cols for _ in range(rows)]` and bounds-check *before* recursing
