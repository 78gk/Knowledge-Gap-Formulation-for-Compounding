# Tweet Thread — Day 1
**Platform:** Twitter/X
**Topic:** Inference-time mechanics — KV cache & prefix caching in multi-turn agents
**Date:** 2026-05-04

---

**Tweet 1 (hook):**
Your LLM agent is probably paying full prompt cost on every single call — even when 90% of the prompt never changes.

One structural fix cuts latency by 85%. Most people never make it because they don't know the rule 🧵

**Tweet 2 (mechanism — two caches):**
There are 2 different caches in LLM inference and they do completely different things:

KV cache → reuses computation WITHIN a single call (automatic, always on)

Prefix cache → reuses computation ACROSS separate calls (only when the prompt prefix is identical)

Mixing them up is the source of the bug.

**Tweet 3 (the rule):**
Prefix caching has one hard rule:

The shared prefix must be bit-for-bit identical between requests.

One volatile field inside your stable system prompt — a prospect name, a confidence score, a timestamp — and the cache is cold.

Every call pays full prefill cost. Every time.

**Tweet 4 (show it — output):**
Measured on a real sales agent prompt:

BROKEN (volatile mixed into stable block):
TechCorp: 847ms | 0 tokens reused
DataFlow: 831ms | 0 tokens reused

FIXED (stable prefix isolated):
TechCorp: 912ms ← fills cache
DataFlow: 134ms | 187 tokens reused ← 85% faster

**Tweet 5 (design rule):**
The fix is positional, not content-based:

✅ Stable block (policy, rules, instructions) → first
✅ Volatile block (prospect data, signals) → appended after

Volatile tokens must never appear inside the stable prefix.
Also: the stable block must be a constant string — same bytes, every call.

**Tweet 6 (link):**
Full explainer with working code + cache hit measurements:
https://kirubel860202.substack.com/p/why-your-llm-agent-pays-full-prompt

Covers: KV cache vs prefix cache, why mixing breaks reuse, the serialization trap, and the multi-block cache pattern for complex agents.

---
*Replace https://kirubel860202.substack.com/p/why-your-llm-agent-pays-full-prompt before publishing.*
