# Research Question — Day 3
**Topic:** Training and post-training mechanics
**Date:** 2026-05-07 (Thursday)
**Partner:** Melaku Yilma

## Question

In Week 11 I fine-tuned Qwen2.5-0.5B-Instruct with LoRA (rank=16, targeting only q_proj and v_proj). The adapter improved overall but consistently failed at the confidence≈0.50 boundary between phrasing tiers — a case the benchmark never caught. How do rank and target-module selection constrain which behavioral corrections are actually representable during fine-tuning, and why does that produce failures near ambiguous boundaries while aggregate scores still look fine?

## Artifact Connection

Week 11 memo (memo.md), Section 4.4 "Disagreement case: TB-0036 — inquiry vs. hypothesis at the boundary" and `training/lora_train.py` lines 36–38 (LORA_RANK=16, LORA_ALPHA=32, LORA_TARGET_MODS=["q_proj","v_proj"]). I shipped the deployment recommendation without being able to explain why the boundary case failed or which config change would address it.

## Why This Gap Matters

Closing this would replace the vague "boundary under-learning" caveat in the memo with a concrete explanation — and give me a specific config change to try if the 14-day reply-rate monitor triggers a rollback.
