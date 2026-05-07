# Research Question — Day 3
**Topic:** Training and post-training mechanics
**Date:** 2026-05-07 (Thursday)
**Partner:** Melaku Yilma

## Question

In Week 11 I trained a LoRA adapter on Qwen2.5-0.5B-Instruct with rank=16, alpha=32, targeting only q_proj and v_proj. I picked those settings from a tutorial — I did not know what any of them actually do. The adapter works well overall, but it gets the phrasing wrong whenever input confidence sits right around 0.50. What do rank, alpha, and target module choice actually control in LoRA training — and which one is most likely causing that boundary failure?

## Artifact Connection

Week 11 memo (memo.md), Section 4.4 "Disagreement case: TB-0036 — inquiry vs. hypothesis at the boundary" and `training/lora_train.py` lines 36–38 (LORA_RANK=16, LORA_ALPHA=32, LORA_TARGET_MODS=["q_proj","v_proj"]). I shipped the deployment recommendation without being able to explain why the boundary case failed or which config change would address it.

## Why This Gap Matters

Closing this would replace the vague "boundary under-learning" caveat in the memo with a concrete explanation — and give me a specific config change to try if the 14-day reply-rate monitor triggers a rollback.
