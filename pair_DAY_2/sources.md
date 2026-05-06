# Sources — Day 2
**Compiled by:** Kirubel Tewodros
**Date:** 2026-05-06 (Wednesday)

## Canonical Papers Read

1. [Efficient Guided Generation for Large Language Models — Willard & Louf (2023)](https://arxiv.org/abs/2307.09702) — establishes the grammar-constrained decoding algorithm: at each token step, a finite-state machine derived from the target schema masks logits so only valid-continuation tokens have non-zero probability. This is the mechanism behind `response_format` and structured outputs.

2. [Structured Generation Improves LLM performance: A Systematic Tag-Constrained Analysis — Tam et al. (2024)](https://arxiv.org/abs/2402.06757) — empirically compares instruction-following vs. constrained decoding for structured output tasks; shows constrained decoding reduces format error rate significantly while introducing ~5-15% latency overhead per call.

## Tool or Pattern Used

- **Tool:** OpenRouter API (direct call experiment)
- **What you ran:** Called the same judge prompt (`judge_prompt.txt`) on 20 edge-case inputs — 10 with instruction-only (no `response_format`), 10 with `response_format={"type": "json_object"}` — using the same DeepSeek V3 model and temperature 0.2
- **What it revealed:** Instruction-only produced preamble text ("Here is the evaluation:") on 3 of 10 edge-case calls. Constrained decoding produced valid JSON on all 10. Latency overhead ~8% per call.

## Follow-on Reading

- [OpenAI Structured Outputs guide](https://platform.openai.com/docs/guides/structured-outputs) — documents the difference between `json_object` mode (valid JSON syntax) and `json_schema` mode (field-level schema enforcement); the distinction matters when field names must also be guaranteed
