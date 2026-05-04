# Sources — Day 1
**Compiled by:** Kirubel Tewodros
**Date:** 2026-05-04

## Canonical Papers Read

1. [Attention Is All You Need](https://arxiv.org/abs/1706.03762) — Vaswani et al., 2017 — Establishes the autoregressive decode mechanism: the transformer generates one token per forward pass, each conditioned on all prior tokens. This structural choice is the root cause of why decode is sequential and why it dominates inference time. Load-bearing in the explainer's mechanism section.

2. [Efficiently Scaling Transformer Inference](https://arxiv.org/abs/2211.05100) — Pope et al., Google, 2022 — Formally names and quantifies the prefill/decode split across model scales and hardware configurations. Shows that on memory-bandwidth-limited hardware (like T4), decode is bound by weight-loading speed rather than arithmetic throughput. Load-bearing in the explainer's explanation of why the T4 decode step costs what it does.

## Tool or Pattern Used
- **Tool:** HuggingFace `transformers` library with `distilgpt2` as a proxy model
- **What you ran:** Timed a single forward pass (`model(**inputs)`) to isolate prefill time, then timed `model.generate()` with `max_new_tokens=64` to get total time. Subtracted to get decode time. Computed per-step cost.
- **What it revealed:** Prefill took 21.4ms (7% of total). Decode took 289.3ms (93% of total) across 64 steps. Per decode step: 4.5ms. This confirmed the mechanism — decode dominates, scales linearly with max_new_tokens, and prefill is negligible regardless of prompt length.

## Follow-on Reading
- [Accelerating Large Language Model Decoding with Speculative Sampling](https://arxiv.org/abs/2302.01318) — Leviathan et al., 2023 — The production technique for attacking the sequential decode bottleneck: a small draft model proposes multiple tokens, the large model verifies them in parallel. 2-3× speedup without quality loss. Direct next step after understanding the prefill/decode split.
