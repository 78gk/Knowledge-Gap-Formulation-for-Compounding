# Partner Brief — Day 2
**For:** Natnael Alemseged
**From:** Kirubel Tewodros
**Topic:** Agent and tool-use internals

---

## My Question (in one sentence)

In my Week 11 pipeline, I told an LLM judge to "return JSON only" via a prompt instruction and parsed the output programmatically — but never enforced a schema. Was that principled or fragile?

## The Real Artifact

`generation_scripts/judge_prompt.txt` + `router_config.json` in Week 11: a DeepSeek/Claude judge is instructed to return JSON with 6 specific fields (`coherence`, `verifiability`, `rubric_clarity`, `mean`, `pass`, `notes`). The output is parsed and filtered at `judge_threshold: 3.5`. No `response_format` parameter — just a prompt instruction.

## What I Think the Answer Is

There are two very different mechanisms behind structured LLM output:
- **Constrained decoding (hard):** At every token step, invalid tokens are masked. The model *cannot* deviate from the schema. This is what `response_format={"type": "json_object"}` or `tool_choice="required"` actually does.
- **Instruction following (soft):** The model learned that "return a JSON score" usually means emit JSON. But it can still produce freeform text — it's just less likely. Without enforcement, this breaks under distribution shift.

My instinct to use deterministic scoring was right — but for a reason I didn't understand at the time.

## Why This Matters for Agent/Tool-Use

This is the same mechanism that makes tool calls in agents unreliable. If you don't enforce `tool_choice="required"`, the model decides probabilistically whether to call the tool. The fix is the same in both cases: enforce at the API level, not the prompt level.

## What I Need From You

- Does your question touch on a similar gap (reliability of agent decisions or structured output)?
- Any pushback: is there a case where prompt-level instruction IS sufficient?

---

*Morning call goal: confirm both questions are sharp, grounded, and resolvable in 600-1000 words.*
