# Product of Array Except Self

`prefix-suffix / array` · **Difficulty:** Medium · **Source:** LC 238 · **Date:** 2026-07-22 · **Companies:** Amazon, Google

**Link:** https://leetcode.com/problems/product-of-array-except-self/

---

## Rating — N/A (concept walk-through, not a timed solve)

> Worked this as a learning session — understood the optimal approach, then drilled the trace and the invariant. No self-attempt to score. The signal that matters: asked *"trace the O(1) case"* and *"what does prefix actually hold"* unprompted — that's the verification + justification-depth habit finally showing up. Keep doing exactly that.

---

## The problem
`res[i]` = product of every element **except** `nums[i]`. No division allowed. Target: O(n) time, O(1) extra space (output array doesn't count).

## Key insight
`res[i]` = (product of everything **left** of `i`) × (product of everything **right** of `i`). Compute those in two sweeps, and fold the right pass directly into the output array so no second array is needed.

## Optimal solution
```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)
        res = [1] * n

        prefix = 1
        for i in range(n):
            res[i] = prefix        # write BEFORE folding in nums[i]
            prefix *= nums[i]

        postfix = 1
        for i in range(n - 1, -1, -1):
            res[i] *= postfix      # res[i] already holds the left product
            postfix *= nums[i]

        return res
```

## The invariant (the thing to say out loud in the interview)
At the instant `res[i]` is written:
- `prefix` = product of everything **strictly before** `i`
- `postfix` = product of everything **strictly after** `i`

The variable earns its name *at the read*, then the next line (`prefix *= nums[i]`) advances it to "everything through `i`," priming it for `i+1`. So it's continuously sliding — it holds what its name says *for one specific instant*.

**Why read-before-update is non-negotiable:** if you did `prefix *= nums[i]` first, `res[i]` would include `nums[i]` itself — the one element it must exclude. That single line ordering is what enforces the "except self."

## Complexity
- **Time:** O(n) — two linear passes
- **Space:** O(1) extra — two scalar accumulators only. The naive prefix[]/suffix[] version is also O(n) time but O(n) space; folding the suffix pass into the output is the downgrade to O(1).

## Verification trace — `[1, 2, 3, 4]` — `[24, 12, 8, 6]`
**Pass 1 (left products, L→R):**

| i | res[i] = prefix | res after | prefix *= nums[i] |
|---|---|---|---|
| 0 | 1 | `[1,1,1,1]` | 1 |
| 1 | 1 | `[1,1,1,1]` | 2 |
| 2 | 2 | `[1,1,2,1]` | 6 |
| 3 | 6 | `[1,1,2,6]` | 24 |

After pass 1: `[1, 1, 2, 6]` (product of everything to the left) ✓

**Pass 2 (right products, R→L):**

| i | postfix before | res[i] *= postfix | res after | postfix *= nums[i] |
|---|---|---|---|---|
| 3 | 1 | 6 | `[1,1,2,6]` | 4 |
| 2 | 4 | 8 | `[1,1,8,6]` | 12 |
| 1 | 12 | 12 | `[1,12,8,6]` | 24 |
| 0 | 24 | 24 | `[24,12,8,6]` | 24 |

Final: `[24, 12, 8, 6]` ✓

Index 2 check: picked up `2` in pass 1 (its left, `1×2`) and `×4` in pass 2 (its right). `2×4 = 8`, and `nums[2]=3` never entered either accumulator when its own slot was written.

## Adversarial case — zeros (the reason division is banned)
`[1, 2, 0, 4]` → `[0, 0, 8, 0]`. Only the zero's own index survives nonzero: its left product `1×2=2` times its right product `4` = 8; every other slot has the zero fold into it. The prefix/suffix method handles one zero (and two zeros → all-zero output) for free, with no special-casing. The division approach `total/nums[i]` dies here — divide-by-zero, and one zero corrupts every index.

## Retro
- ✅ **Went well:** drove the session toward the trace and the invariant without being told — the exact verification + justification-depth habit that was missing on the previous three problems
- 🔁 **Pattern trigger** — "each element depends on everything-but-itself" (products, sums, counts excluding self) → **two-pass prefix/suffix**, folding the second pass into the output for O(1) space. Always: write the accumulator into the slot *before* folding the current element in
- ⚠️ **Follow-up to expect:** "now do it without a second array" — this O(1) version *is* the answer; and "what about division?" → zeros + the explicit no-division constraint
