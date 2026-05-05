# Morning Call Summary — Day 1
**Partners:** Kirubel & Nurye
**Date:** 2026-05-04 (Monday)

## What Was Ambiguous in the Original Drafts
Kirubel's original draft asked three things at once: whether 256 tokens takes twice as long as 128, what the model is doing during 320ms, and what to change for 2x speed. Nurye pushed back: "You're asking three different questions. Which mechanism are you actually confused about — why the time is what it is, or how to change it?" Kirubel also had not named which phase of inference (prefill vs decode) was the specific gap — the question described a symptom (320ms) without naming the underlying mechanism split.

## How Each Question Was Sharpened
Nurye's interrogation surfaced that the real gap was not "how do I optimize" but "I don't know which phase dominates and therefore can't reason about the number at all." The question was rewritten to name the prefill/decode split explicitly as the mechanism being asked about, remove the optimization sub-questions, and anchor to the specific claim in the memo ("negligible latency overhead") that this gap undermines. The question went from three loose sub-questions to one precise mechanism question.

## Final Committed Questions
- Kirubel's question: Is the ~320ms per task in my Week 11 evaluator dominated by prefill (reading the prompt in parallel) or decode (generating 256 output tokens sequentially), and what does that tell me about whether my max_new_tokens=256 choice was reasonable?
- Nurye's question: In TheConversionEngine, every call to `generation_service.draft_email_from_scaffold` passes a large prompt mixing stable policy instructions with volatile prospect signals. The traces show aggregate latency but no breakdown. Is the repeated stable content actually being reused by the provider's prefix cache, or re-ingested from scratch on every call — and which prompt assembly decisions determine whether caching kicks in at all?

*Confirmed by both partners: [x] Yes*
