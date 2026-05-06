# Research Question — Day 2
**Topic:** Agent and tool-use internals
**Date:** 2026-05-06 (Wednesday)
**Partner:** Natnael Alemseged

## Question

In my Week 11 pipeline, I told an LLM judge to "return JSON only" — and it usually did. But when it didn't, my pipeline broke silently. Does telling a model to return JSON actually *force* it to, or does it just make JSON more likely? What is the real mechanism, and what does it mean for any system that acts on structured LLM output?

## Artifact Connection

Week 11 — `generation_scripts/judge_prompt.txt` and `router_config.json`: the LLM judge is instructed to return JSON with fields `coherence`, `verifiability`, `rubric_clarity`, `mean`, `pass`, and `notes`. The output is parsed and filtered at `judge_threshold: 3.5`. No `response_format` parameter or grammar enforcement was used — just a prompt instruction.

## Why This Gap Matters

If instruction-following is just probability (not a hard constraint), then any prompt or context shift could cause the judge to return freeform text, silently break the JSON parse, and corrupt the dataset construction. Closing this gap means I can say precisely whether the pipeline needed `response_format={"type": "json_object"}` to be reliable — and the same answer applies to every agent that acts on structured LLM output.
