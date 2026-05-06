# Evening Call Summary — Day 2
**Partners:** Kirubel Tewodros & Natnael Alemseged
**Date:** 2026-05-06 (Wednesday)

## Feedback You Gave Natnael on His Explainer

Natnael's explainer on the ReAct stopping signal was clear on what the agent does but underexplained *why* it stops — the mechanism (end-of-turn token trained into the model via RLHF) wasn't named until the third paragraph. The opening hooked on a relatable failure (agent looping forever) but then jumped to solution before establishing the mechanism. Specific note: the sentence "the model decides when it's done" is true but does no explanatory work — pushed him to replace it with the actual mechanism sentence. The code example was strong.

## Feedback You Received on Your Explainer

Natnael's main push: the explainer explains constrained decoding well but doesn't answer the implicit follow-up — does `json_object` mode guarantee correct field *names*, or just valid JSON syntax? He was right. I had glossed over this. Valid JSON and schema-conformant JSON are different things — `json_object` gives you valid JSON, `response_format={"type": "json_schema", "json_schema": {...}}` gives you field-level enforcement. He also said the opening bug description was the strongest part and the fix code block should have come earlier.

## Revisions Made After the Call

1. Added one clarifying sentence to the constrained decoding section: `json_object` guarantees valid JSON syntax, not field names — for field-level enforcement, use `json_schema` mode with an explicit schema.
2. Moved the fix code block to appear right after the two-mechanism section, before the tool-calling analogy — makes the payoff land earlier.

*Written by: [x] Kirubel Tewodros*
*Confirmed by other partner: [x] Yes*
