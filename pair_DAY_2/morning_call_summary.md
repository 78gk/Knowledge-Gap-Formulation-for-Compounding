# Morning Call Summary — Day 2
**Partners:** Kirubel Tewodros & Natnael Alemseged
**Date:** 2026-05-06 (Wednesday)

## What Was Ambiguous in the Original Drafts

Kirubel's draft question conflated two different gaps: (1) understanding *why* a prompt instruction doesn't force JSON at the mechanism level, and (2) understanding what the pipeline should do differently once you know that. Natnael pushed back: those are different questions — one is about model behavior, the other is system design. The draft also didn't specify what actually happens when the judge returns non-JSON — whether the pipeline throws a caught error or silently corrupts the downstream decision.

Natnael's draft question was about what determines when a ReAct agent stops calling tools and emits a final answer — also unclear on whether it was asking about the learned stopping signal or the prompt engineering that shapes it.

## How Each Question Was Sharpened

Natnael asked three things that forced precision:
1. "Which gap feels more real — the mechanism or the fix?"
   → Kirubel committed to the mechanism. The fix (one API parameter) is trivial once the mechanism is understood.
2. "What actually happens when the judge returns non-JSON right now?"
   → Kirubel confirmed: `json.loads()` raises `JSONDecodeError`, the exception is not caught, the task is silently dropped from the dataset construction pipeline.
3. "Are you asking about model behavior or system design?"
   → Model behavior. The system design question answers itself once the mechanism is clear.

Natnael's question was sharpened similarly: from "why does the agent get stuck" to "what is the learned stopping signal in a ReAct loop — is it a trained behavior or a prompt-engineered threshold?"

## Final Committed Questions

- **Kirubel:** In my Week 11 pipeline, I told an LLM judge to "return JSON only" — and it usually did. But when it didn't, my pipeline broke silently. Does telling a model to return JSON actually *force* it to, or does it just make JSON more likely? What is the real mechanism, and what does it mean for any system that acts on structured LLM output?
- **Natnael:** In my Week 10 ReAct agent, the model sometimes looped through the same tool call multiple times before stopping. What actually determines when the model emits a final answer instead of another tool call — is it a trained stopping signal, a prompt threshold, or something else?

*Confirmed by both partners: [x] Yes*
