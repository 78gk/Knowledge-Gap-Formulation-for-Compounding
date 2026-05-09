# Gap Closure Sign-off — Day 5
**Asker:** Kirubel Tewodros
**Explainer written by:** Bereket Haile (for Kirubel's speculative decoding question)
**Date:** 2026-05-09 (Saturday)

## Gap Closure Judgment

[x] Closed

## What I Understand Now That I Didn't Before

Before today, "speculative decoding" was a technique I had seen referenced in production optimization discussions without understanding the mechanism. I now understand that speculative decoding uses a small draft model to generate k candidate tokens speculatively, then verifies all k in parallel with a single forward pass of the target model. The acceptance criterion uses rejection sampling: each draft token is accepted with probability min(1, p_target(token) / p_draft(token)), preserving the target model's output distribution exactly. The speed gain comes from the fact that one target-model forward pass can verify k tokens simultaneously — which is possible because the target model's prefill is parallelizable, even though its decode is not.

The critical insight for my evaluator: speculative decoding requires the target model to be significantly larger than the draft model, because the draft model must be cheap enough that k draft steps plus one target verification step is faster than k sequential target-model steps. For my 0.5B Qwen evaluator, there is no practical draft model — 0.5B is already small. The technique is most valuable for 7B+ target models with a 125M or 350M draft. For a 0.5B evaluator, the right optimization is exactly what Day 1 analysis identified: reduce max_new_tokens from 256 to 64. That is structurally equivalent to speculative decoding's goal (fewer target-model forward passes) but without the draft-model overhead. Understanding speculative decoding's regime makes this Day 1 conclusion defensible rather than coincidental.

## What Remains Open

One sub-question: multi-token prediction (MTP) architectures — where the model itself is trained to predict k tokens ahead — may eventually replace the draft-model paradigm for small models. Not relevant for the current evaluator but worth tracking for future inference optimization.
