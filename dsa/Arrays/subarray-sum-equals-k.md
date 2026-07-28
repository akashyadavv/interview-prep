# Subarray Sum Equals K

`prefix-sum / hashmap` · **Difficulty:** Medium · **Source:** LC 560 · **Date:** 2026-07-22 · **Companies:** Amazon, Google

**Link:** https://leetcode.com/problems/subarray-sum-equals-k/

---

## Rating — N/A (concept session)

> Walked the approach rather than a timed cold solve. But the reasoning was strong and self-driven: correctly diagnosed **why two pointers fails** (negatives break window monotonicity) *before* asking for the answer, and framed the prefix-sum complement idea independently. That's exactly the "state the tradeoff out loud" signal interviewers reward.

---

## Why two pointers / sliding window fails here
Sliding window relies on **monotonicity**: expanding the window only grows the sum, shrinking only shrinks it — that's what makes "sum too big → move left" a valid decision. With **negative numbers**, expanding a window can *decrease* the sum, so "too big" no longer reliably means "shrink." The invariant collapses. Negatives on the table → sliding window is out.

## The insight
```
prefix[i] - prefix[j] == k     (subarray (j, i] sums to k)
→ prefix[j] == prefix[i] - k
```
So at each `i`, ask: **"how many earlier prefix sums equal `prefix[i] - k`?"** That's Two Sum's complement trick on prefix sums — "have I seen the thing that completes me?" A hashmap of `prefixSum → count` answers it in O(1), killing the nested `j` loop.

## Solution
```python
from collections import defaultdict

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefixCount = defaultdict(int)
        prefixCount[0] = 1              # empty prefix seen once, before the array
        currentSum, result = 0, 0

        for num in nums:
            currentSum += num
            result += prefixCount[currentSum - k]   # look up BEFORE inserting
            prefixCount[currentSum] += 1

        return result
```

## The two lines that carry the whole problem

**`prefixCount[0] = 1` — the seed.** "A prefix sum of 0 occurred once, before we started." Without it you miss every subarray that starts at index 0. E.g. `nums=[3], k=3`: after the loop `currentSum=3`, look up `3-3=0` — if `0` isn't seeded you find nothing and return 0 instead of 1. The empty-prefix seed lets a whole-prefix subarray count itself.

**Lookup before insert.** `result += ...` runs *before* `prefixCount[currentSum] += 1`. If you inserted first, then when `k == 0` you'd match the current prefix against itself and count an empty subarray. Lookup-then-insert guarantees every matched `j` is strictly earlier than `i`.

> Minor gotcha with `defaultdict`: reading `prefixCount[currentSum - k]` *inserts* that key with value 0 if absent — harmless (count 0 adds nothing) but it does grow the map with zero-count keys. `.get(key, 0)` on a plain dict avoids the side effect. Not a bug; worth knowing you noticed.

## Complexity
- **Time:** O(n) — single pass, O(1) map ops per element
- **Space:** O(n) — worst case every prefix sum is distinct

## Verification trace — negatives (the whole reason we're not using two pointers)
`nums = [1, -1, 1]`, `k = 1`. Expected **3**: `[1]`@0, `[1]`@2, `[1,-1,1]` whole.

| num | currentSum | look up `sum−k` | in map? | result | map after |
|---|---|---|---|---|---|
| start | 0 | — | — | 0 | `{0:1}` |
| 1 | 1 | 0 | yes ×1 | 1 | `{0:1, 1:1}` |
| −1 | 0 | −1 | no | 1 | `{0:2, 1:1}` |
| 1 | 1 | 0 | yes ×2 | 3 | `{0:2, 1:2}` |

Last step: `sum−k = 0` is in the map **twice** (before the array, and after `[1,-1]` cancels), so it adds 2 — catching both `[1]`@2 and the full `[1,-1,1]`. Storing **counts**, not just presence, is what makes overlapping subarrays on the same prefix all tally. ✅

## Retro
- ✅ **Went well:** diagnosed the negatives-break-monotonicity tradeoff unprompted; derived the complement rearrangement independently; used a clean `defaultdict` idiom
- 🔑 **Pattern trigger** — "count/find **subarrays** with a target sum" (esp. with negatives) → **prefix sum + hashmap of counts**, seeded `{0:1}`, look up `prefix - k` before inserting. This is Two Sum's complement on prefixes
- ⚠️ **Variant to drill next:** **Contiguous Array (LC 525)** — same trick, but map `0 → -1` to convert "equal 0s and 1s" into "subarray sums to 0." Also **LC 974** (subarray sums divisible by k) uses `prefix % k` as the key
