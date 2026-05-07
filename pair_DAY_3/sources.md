# Sources — Day 3
**Compiled by:** Kirubel Tewodros
**Date:** 2026-05-07 (Thursday)

## Canonical Papers Read

1. [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685) (Hu et al., 2021) — defines the rank decomposition ΔW = B·A, the α scaling convention, and the original guidance on target module selection (q_proj, v_proj as defaults)

2. [Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning](https://arxiv.org/abs/2012.13255) (Aghajanyan et al., 2021) — shows that standard fine-tuning tasks have intrinsic dimensionality as low as 200, explaining why low-rank adapters achieve strong benchmark scores without capturing fine-grained boundary behavior

3. [LoRA Learns Less and Forgets Less](https://arxiv.org/abs/2405.09673) (Biderman et al., 2024) — empirical analysis showing rank-limited adapters systematically under-perform on distribution-shifted evaluation while preserving in-distribution scores

## Tool Used
- **Tool:** Claude Code with direct Week 11 artifact access
- **What you ran:** Read `training/lora_train.py` (lines 36–38), `memo.md` Section 4.4, and `ablation_results.json` to ground the explainer in the actual boundary failure (TB-0036, conf=0.55)
- **What it revealed:** The rank=16, alpha=32, q+v-only config was chosen by convention with no mechanistic justification; the memo names "boundary under-learning" as a caveat but does not explain it — confirming the gap was genuine and unresolved at the time of submission
