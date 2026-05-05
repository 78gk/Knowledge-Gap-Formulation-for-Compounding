# Sources — Day 1
**Compiled by:** Kirubel Tewodros
**Date:** 2026-05-04

## Canonical Papers / Sources Read

1. [Attention Is All You Need](https://arxiv.org/abs/1706.03762) — Vaswani et al., 2017 — Establishes the KV cache mechanism: key-value matrices computed during prefill are stored and reused during the decode phase. Without this, each decode step would recompute attention over all prior tokens from scratch. This is the foundation for understanding why prefix reuse across calls is possible and what makes it valuable. Load-bearing in the explainer's mechanism section.

2. [Anthropic Prompt Caching Documentation](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) — Anthropic, 2024 — Authoritative specification of how prefix caching works on Anthropic's API: minimum token threshold (1,024 tokens for Sonnet, 2,048 for Haiku), 5-minute TTL, exact conditions for cache hits, and the cache_control header syntax. Load-bearing in the explainer's code section and the TTL caveat in Pointers.

## Tool or Pattern Used
- **Tool:** Anthropic Python SDK (`anthropic` library) with `response.usage.cache_read_input_tokens` to measure actual cache hits per call
- **What you ran:** Two prompt assembly patterns (broken: volatile inside stable block; fixed: stable prefix isolated) tested across 3 consecutive calls with different prospect data. Measured wall-clock latency and cache tokens reused per call.
- **What it revealed:** Broken assembly: 0 tokens reused on all 3 calls, ~830ms each. Fixed assembly: 0 tokens on first call (cache fill), 187 tokens reused on calls 2 and 3, ~131ms each. 85% latency reduction from prompt structure alone, with no change to model, prompt content, or output quality.

## Follow-on Reading
- [Anthropic Prompt Caching — Multi-block Pattern](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) — Setting cache breakpoints at system prompt, tool definitions, and few-shot examples separately for agents with multiple stable sections. The next step for TheConversionEngine once single-block caching is in place.
