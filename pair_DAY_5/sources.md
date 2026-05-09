# Sources — Day 5
**Compiled by:** Kirubel Tewodros
**Date:** 2026-05-09 (Saturday)

## Canonical Papers Read

1. [Fast Inference from Transformers via Speculative Decoding](https://arxiv.org/abs/2211.17192) (Leviathan et al., ICML 2023) — original speculative decoding paper; proves the rejection sampling acceptance criterion preserves the target model's output distribution exactly; benchmarks the speedup as a function of draft model acceptance rate and k (number of speculative tokens)

2. [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/abs/2305.18290) (Rafailov et al., NeurIPS 2023) — derives the DPO objective; shows it is mathematically equivalent to RLHF with a specific reward parameterization without requiring a separate reward model or RL training loop; load-bearing for the explainer's training objective comparison

3. [ORPO: Monolithic Preference Optimization Without Reference Model](https://arxiv.org/abs/2403.07691) (Hong et al., 2024) — introduces the odds-ratio penalty for single-pass SFT + alignment; directly compares to DPO on Llama-2 and Phi-2 benchmarks; the "no reference model" property is the key practical difference

## Tool Used
- **Tool:** Claude Code with direct Week 11 artifact access
- **What you ran:** Re-read `run_sealed_ablation.py` decode loop and `memo.md` §9.2 to confirm the 0.5B target model profile; cross-referenced speculative decoding speedup conditions (target model size, draft model availability) against this profile to verify non-applicability
- **What it revealed:** At 0.5B parameters, the evaluator already operates in the "draft model" regime — there is no practical smaller model to use. The technique's non-applicability is structural, not a configuration choice. This confirms the Day 1 max_new_tokens reduction as the correct and only lever available in this setup.

## Follow-on Reading
- [Medusa: Simple LLM Inference Acceleration Framework with Multiple Decoding Heads](https://arxiv.org/abs/2401.10774) (Cai et al., 2024) — alternative to draft-model speculative decoding that adds prediction heads directly to the target model; potentially applicable to smaller models since it requires no separate draft model. Worth evaluating if the evaluator is ever retrained at rank=64 with k_proj.
