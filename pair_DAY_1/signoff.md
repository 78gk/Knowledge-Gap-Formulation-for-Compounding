# Gap Closure Sign-off — Day 1
**Asker:** Nurye Nigus Mekonen
**Explainer written by:** Kirubel Tewodros
**Date:** 2026-05-04

## Gap Closure Judgment
[x] Closed

## What I Understand Now That I Didn't Before
Before this explainer I could see aggregate latency in the traces but had no way to reason about whether the stable prompt content in TheConversionEngine was being reused. I now understand that there are two separate caching mechanisms — KV cache (within a call, always on) and prefix caching (across calls, conditional on identical byte sequences) — and that my prompt assembly was likely breaking prefix caching by embedding volatile prospect fields inside the stable policy block. The fix is positional: stable content must form an unbroken prefix, with volatile content appended strictly after. I also now understand that the Anthropic prefix cache has a 5-minute TTL, which matters for agents with low call frequency. The code output showing 0 tokens reused vs 187 tokens reused made this concrete — I can go back to TheConversionEngine and restructure the prompt assembly immediately.

## What Remains Open
The multi-block cache pattern (setting cache breakpoints at system prompt, tool definitions, and few-shot examples separately) was mentioned but not demonstrated — that is a follow-on worth researching for v2 of the agent.
