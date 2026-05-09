# Gap Closure Sign-off — Day 5
**Asker:** Kirubel Tewodros
**Format:** Self-assessment + AI critique (paired-research portion of program concluded with Day 4)
**Date:** 2026-05-09 (Saturday)

## Gap Closure Judgment

[x] Closed (speculative decoding question)
[x] Closed (DPO/ORPO companion question)

## What I Understand Now That I Didn't Before

**On speculative decoding.** Before today, "speculative decoding" was a technique I had seen referenced in production optimization discussions without understanding the mechanism. I now understand that speculative decoding uses a small draft model to generate k candidate tokens speculatively, then verifies all k in parallel with a single forward pass of the target model. The acceptance criterion uses rejection sampling: each draft token is accepted with probability min(1, p_target(token) / p_draft(token)), preserving the target model's output distribution exactly. The speed gain comes from the fact that one target-model forward pass can verify k tokens simultaneously — which is possible because the target model's prefill is parallelizable, even though its decode is not.

The critical insight for my evaluator: speculative decoding requires the target model to be significantly larger than the draft model, because the draft model must be cheap enough that k draft steps plus one target verification step is faster than k sequential target-model steps. For my 0.5B Qwen evaluator, there is no practical draft model — 0.5B is already small. The technique is most valuable for 7B+ target models with a 125M or 350M draft. For a 0.5B evaluator, the right optimization is exactly what Day 1 analysis identified: reduce max_new_tokens from 256 to 64. That is structurally equivalent to speculative decoding's goal (fewer target-model forward passes) but without the draft-model overhead. Understanding speculative decoding's regime makes this Day 1 conclusion defensible rather than coincidental.

**On DPO vs SFT.** Writing the companion explainer surfaced that DPO and SFT solve mechanistically different problems. SFT maximizes the likelihood of demonstrations and provides no signal about what to avoid; DPO optimizes a preference (chosen over rejected, relative to a reference model) and its gradient fires hardest exactly where SFT's gradient is silent — at the boundary between outputs with similar prior probability. The diagnostic that emerged: if a model failure is representational (rank too low to express the distinction), DPO cannot fix it; if it is preferential (capacity exists but no contrast signal was given), DPO is the right tool. Combined with Day 3's rank-geometry analysis, this gives a complete diagnostic ordering for the boundary failure: fix the rank ceiling first; if failures persist, reach for preference data and DPO.

## What Remains Open

For speculative decoding: multi-token prediction (MTP) architectures — where the model itself is trained to predict k tokens ahead — may eventually replace the draft-model paradigm for small models. Not relevant for the current evaluator but worth tracking. For DPO: the question of how β controls the trade-off between alignment strength and reference-model deviation in production settings is a related gap I have not yet researched.
