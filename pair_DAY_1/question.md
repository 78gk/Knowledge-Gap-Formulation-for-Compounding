# Research Question — Day 1
**Topic:** Inference-time mechanics
**Date:** 2026-05-04 (Monday)
**Partner:** Nurye

## Question
In my Week 11 memo I measured ~320ms per task (greedy decode, max_new_tokens=256, T4 GPU) and reported this as evidence of "negligible latency overhead." But I set max_new_tokens=256 without knowing what drives that 320ms. Is that time dominated by the prefill phase — the model reading my input prompt once in parallel — or the decode phase — the model generating each of the 256 output tokens one at a time sequentially? The answer changes whether my parameter choice was reasonable and what I would actually tune to make the evaluator faster.

## Artifact Connection
Week 11 memo (memo.pdf), Section 9.2 Cost-Pareto: "Latency measured as median wall-clock per-task inference time (greedy decode, max_new_tokens=256) across the 62 held-out tasks." I published this number and this parameter choice without being able to explain either.

## Why This Gap Matters
Closing this would let me replace a number I stated blindly with a real cost model — and tell me exactly which parameter to change if the evaluator needed to run faster at scale.
