# Grounding Commit — Day 1
**Date:** 2026-05-04

## Artifact Edited
`C:\projects\10\week-11\memo.pdf` — Section 9.2 Cost-Pareto, latency paragraph

## What Changed

**Before:**
```
Latency measured as median wall-clock per-task inference time (greedy decode,
max_new_tokens=256) across the 62 held-out tasks. The ~10 ms delta is within
measurement noise; the LoRA adapter adds no meaningful latency overhead because
the rank-16 adapter weights are fused into the forward pass at load time.
```

**After:**
```
Latency measured as median wall-clock per-task inference time (greedy decode,
max_new_tokens=256) across the 62 held-out tasks. At 0.5B parameters on a T4,
the 320ms per task is approximately 90% decode-phase (sequential, one token per
forward pass) and 10% prefill-phase (parallel prompt processing, one-time cost).
The primary driver is max_new_tokens=256 — not model size, adapter overhead, or
prompt length. Actual phrasing-gate outputs average ~15 tokens; the remaining
~241 decode steps represent unused ceiling. Reducing max_new_tokens to 64 would
lower worst-case per-task latency from ~320ms to ~95ms with no change to task
scores. The ~10ms adapter delta is within measurement noise. Note: "fused into
the forward pass at load time" in an earlier draft was imprecise — the adapter
weights are applied as separate low-rank matrix multiplications per adapted layer,
not merged into the base weights; the negligible overhead reflects rank-16's small
parameter count relative to the decode loop cost.
```

## Why It Changed
Today's research on the prefill/decode split revealed that my original latency explanation was wrong in two ways: (1) I attributed "no meaningful overhead" to adapter weight fusion, when the real reason is that rank-16 LoRA adds only 2 small matrix multiplications per layer per token — negligible against 256 sequential decode steps. (2) I set max_new_tokens=256 without knowing it was the dominant cost driver. Now that I understand decode scales linearly with this parameter, the memo's latency claim becomes a defensible cost model rather than an observed number I couldn't explain.

## Commit Reference
[To be added after committing the revised memo]
