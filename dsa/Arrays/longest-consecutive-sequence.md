# Longest Consecutive Sequence

`hashset / array` · **Difficulty:** Medium · **Source:** LC 128 · **Date:** 2026-07-22 · **Companies:** Amazon, Google

**Link:** https://leetcode.com/problems/longest-consecutive-sequence/

---

## Rating — 3/5

| # | Signal | Score | Note |
|---|--------|:-----:|------|
| 1 | Explained approach out loud *before* coding | 3/5 | Discussed the set approach conceptually, but the start-guard logic was wrong going in |
| 2 | Reached optimal (or justified the tradeoff) | 3/5 | Got to O(n), but only after the inverted guard was corrected; first attempt was O(n log n) sort |
| 3 | Code was clean & bug-free on first pass | 2/5 | The O(n) attempt had an inverted guard → correct output but O(n²). Sneakiest kind of bug: right answer, wrong complexity |
| 4 | Complexity analysis correct & fast | 4/5 | Stated O(n)/O(n) correctly |
| 5 | **Verified — caught the problem by running** | 4/5 | **Self-caught the TLE by running it, before asking.** First time this session verification fired at the right moment. Hand-trace still didn't come voluntarily |

**Trend note:** best *habits* of the session. The verification instinct finally showed up unprompted — running the code and catching the TLE is exactly the move. Next lever: run the trace *before* submitting, so you catch the inverted guard on paper instead of via TLE.

---

## The trap this problem sets
The obvious solution is **sort → scan for runs** (my first attempt below). It works and is clean, but it's O(n log n). The entire point of LC 128 is that **O(n) is achievable**, and the guaranteed interviewer follow-up is "can you do it without sorting?" Reaching for sort walks straight into the trap.

## Attempt 1 — sort (works, but suboptimal)
```python
def longestConsecutive(self, nums):
    if not nums: return 0
    nums.sort()
    count = max_length = 1
    for i in range(1, len(nums)):
        if nums[i] == nums[i-1]:      # dedupe — nice touch, avoids double-count
            continue
        elif nums[i] - nums[i-1] == 1:
            count += 1
            max_length = max(max_length, count)
        else:
            count = 1
    return max_length
```
Correct (handles empty + duplicates cleanly). **O(n log n)** — dominated by the sort.

## Attempt 2 — the TLE bug (inverted guard)
```python
for x in nums_set:
    if x - 1 in nums_set:            # ← WRONG: starts from every NON-start
        count = 1
        i = x
        while i - 1 in nums_set:     # ← and walks DOWNWARD
            i -= 1; count += 1
        max_length = max(max_length, count)
```
**Why it's O(n²):** launching a walk from every element that *has* a predecessor means every element except the start re-walks the run beneath it. Run `[1,2,3,4]` → `4+3+2+1` steps. One long block of n → n² → **TLE**. Output is still correct, which is what makes a complexity bug so easy to ship.

## Corrected O(n) solution
```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if not nums:
            return 0
        nums_set = set(nums)
        max_length = 0
        for x in nums_set:
            if x - 1 not in nums_set:        # only start from a sequence START
                count = 1
                i = x
                while i + 1 in nums_set:     # walk UP the run
                    i += 1
                    count += 1
                max_length = max(max_length, count)
        return max_length
```
Two flips from attempt 2: guard `x-1 not in` (start only), and walk `i+1` (upward).

## Why the fixed version is O(n) despite a loop-in-a-loop (the interview signal)
The inner `while` only ever launches from a **true sequence start**. So across the whole outer loop, the inner walk touches each element **at most once** — e.g. `5` in `[3,4,5,6]` is only walked when we start from `3`; on its own turn `5` is skipped instantly because `4` (its predecessor) is in the set. Total inner steps summed over all starts = n → **O(n)**. Attempt 2 failed exactly because *every* element could launch a walk, so there was no "at most once."

## Set operations used
- `set(nums)` — build, O(n), dedupes for free
- `x - 1 not in nums_set` — is `x` a start? O(1)
- `while i + 1 in nums_set` — extend the run, O(1) per step
- Iterate the **set**, not the list — each distinct value considered once (matters under heavy duplicates)

## Complexity
- **Time:** O(n) — set build O(n) + each element touched at most once by an inner walk
- **Space:** O(n) — the set

## Verification trace — `[100,4,200,1,3,2]`, set `{1,2,3,4,100,200}` → expected 4
| x | `x-1` in set? | action | run | max |
|---|---|---|---|---|
| 100 | no | start; 101? no | 1 | 1 |
| 4 | yes (3) | skip | — | 1 |
| 200 | no | start; 201? no | 1 | 1 |
| 1 | no | start; walk 2→3→4, 5? no | **4** | 4 |
| 3 | yes (2) | skip | — | 4 |
| 2 | yes (1) | skip | — | 4 |

The skips on 4/3/2 are the efficiency story — they never launch a walk. Result **4** ✅

## Retro
- ✅ **Went well:** first attempt correct with clean dedupe/empty handling; **caught the TLE by running the O(n) attempt myself** — verification fired at the right time for the first time this session
- ❌ **Went wrong:** inverted the start-guard (walked from non-starts, downward) → correct answer but O(n²); a hand-trace before submitting would have caught it on paper. Also reached for sort first, which is the trap this problem is built around
- 🔑 **Pattern trigger** — "longest run of consecutive integers, order doesn't matter" → **hash set + start-only walk**: dump to a set, only begin counting from `x` where `x-1 ∉ set`, walk upward. The `x-1 not in set` guard is what makes the nested loop O(n) — be ready to *explain why*, that's the whole interview point
