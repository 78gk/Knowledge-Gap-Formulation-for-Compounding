# Tweet Thread — Day 1
**Platform:** Twitter/X
**Topic:** Inference-time mechanics
**Date:** 2026-05-04

---

**Tweet 1 (hook):**
I published a latency number in my ML paper — 320ms per eval task — without knowing what was driving it.

Turns out I could cut it by 70% by changing one parameter I set blindly.

The mechanism behind it is something every LLM practitioner needs to understand 🧵

**Tweet 2 (mechanism — prefill):**
Every LLM inference call has 2 phases with completely different performance profiles.

Phase 1 — Prefill:
The model reads your entire input prompt in ONE parallel pass.
All tokens processed simultaneously on GPU.
Fast. Scales well. Usually 10-20ms even for long prompts.

**Tweet 3 (mechanism — decode):**
Phase 2 — Decode:
The model generates output tokens ONE AT A TIME.
Token N requires token N-1. No parallelism possible.
This is structural — it's how autoregressive transformers work.

Each step = ~1-5ms on a T4.
256 steps = ~256-1280ms.

Decode dominates your inference time.

**Tweet 4 (show it — code output):**
I profiled this on a small proxy model:

```
Input tokens:        71
Prefill (parallel):  21.4ms   → 7% of total
Decode (64 steps):   289.3ms  → 93% of total
Per decode step:     4.5ms
```

Prefill is a one-time cost. Decode scales linearly with max_new_tokens.

**Tweet 5 (adjacent concept + FDE implication):**
Why isn't decode even slower? The KV cache.

Key-value matrices from prefill are stored and reused every decode step — no recomputation.

Production implication: My eval tasks output ~15 tokens on average. I set max_new_tokens=256.
Cutting to 64 → same scores, 3× faster. Always check your actual output length distribution.

**Tweet 6 (link):**
Full explainer with timing code + breakdown of the KV cache role:
INSERT_BLOG_URL

Covers: why 320ms is 93% decode, what max_new_tokens actually controls, and how speculative decoding attacks this bottleneck in production.

---
*Replace INSERT_BLOG_URL before publishing.*
