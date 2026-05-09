# Research Question — Day 5
**Topic:** Production patterns / Inference optimization
**Date:** 2026-05-09 (Saturday)
**Partner:** Bereket Haile

## Question

In my Week 11 evaluator, Day 1 research revealed that ~90% of the 320ms per-task latency is in the sequential decode phase — 256 forward passes, one token at a time. I have since seen references to "speculative decoding" as a production technique that speeds up decode without changing output distribution. But I don't know what the mechanism is, what a "draft model" is, how the acceptance criterion works, or whether the technique would even apply to my setup: a 0.5B model running greedy decode on a T4. Does speculative decoding address my evaluator's bottleneck, and if not, what does understanding it tell me about when it is and isn't the right lever to reach for?

## Artifact Connection

Week 11 `run_sealed_ablation.py` and `memo.md` Section 9.2 Cost-Pareto. The latency profile (320ms/task, 256 decode steps, Qwen2.5-0.5B-Instruct on T4) is fully characterized from Day 1 research. The question is whether speculative decoding changes the deployment options for this specific setup — or whether it explains why my existing choice (reduce max_new_tokens) was already the right optimization.

## Why This Gap Matters

If speculative decoding applies to small models, it changes the cost profile of the evaluator at production scale. If it doesn't, understanding why gives me a principled framework for knowing which inference optimization techniques map to which hardware/model-size regimes — a gap that will recur every time I build a new evaluator or deploy a new model.
