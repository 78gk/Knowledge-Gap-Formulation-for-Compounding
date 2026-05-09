# Grounding Commit — Day 5
**Date:** 2026-05-09 (Saturday)

## Artifact Edited

`week-11/memo.md` — Section 9.2 Cost-Pareto, inference optimization note (appended after Day 1 grounding edit)

## What Changed

**Before (after Day 1 edit, before Day 5):**
```
Latency measured as median wall-clock per-task inference time (greedy decode,
max_new_tokens=256) across the 62 held-out tasks. At 0.5B parameters on a T4,
the 320ms per task is approximately 90% decode-phase (sequential, one token per
forward pass) and 10% prefill-phase (parallel prompt processing, one-time cost).
The primary driver is max_new_tokens=256 — not model size, adapter overhead, or
prompt length. Actual phrasing-gate outputs average ~15 tokens; the remaining
~241 decode steps represent unused ceiling. Reducing max_new_tokens to 64 would
lower worst-case per-task latency from ~320ms to ~95ms with no change to task
scores.
```

**After:**
```
[same paragraph, with appended note:]

Inference optimization note: speculative decoding — a production technique that
verifies k draft-model tokens in one target-model forward pass — does not apply
to this evaluator. The technique requires a target model large enough that one
forward pass verifying k draft tokens is faster than k sequential target steps.
At 0.5B parameters, this condition does not hold: there is no lighter draft model
that is cheap enough to make the math work. The Day 1 recommendation (reduce
max_new_tokens) achieves the same goal — fewer target-model forward passes — by
removing unused decode ceiling rather than by draft-model speculation. For future
evaluators using 7B+ models, speculative decoding with a 125M–350M draft model
would be the right inference optimization to evaluate first.
```

## Why It Changed

Before today I had the Day 1 recommendation (reduce max_new_tokens) but no principled explanation for why inference optimization techniques like speculative decoding did not apply. Understanding speculative decoding's regime — large target model, cheap draft model, parallel verification — provides the mechanistic reason: at 0.5B, the evaluator already IS the small model. The grounding edit makes the original optimization recommendation more defensible by explaining the design space it sits in.

## Commit Reference

`5589bd2` — week-11 repo (main branch)
