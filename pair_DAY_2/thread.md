# Tweet Thread — Day 2
**Platform:** Twitter/X
**Topic:** Agent and tool-use internals — structured output mechanism
**Date:** 2026-05-06 (Wednesday)

---

**Tweet 1 (hook):**
My Week 11 pipeline broke because an LLM judge returned "Here is my evaluation: {...}" instead of raw JSON.

I "fixed" it by adding "respond only in JSON" to the system prompt.

I thought that was the real fix.

It wasn't.

🧵

---

**Tweet 2 (the two mechanisms):**
There are two completely different ways to get structured output from an LLM:

1. Instruction following — the model learned to usually comply
2. Constrained decoding — the model is physically blocked from producing invalid tokens

Only one of these actually works.

---

**Tweet 3 (constrained decoding explained):**
Constrained decoding works like this:

At each token step, a grammar masks out any token that would break the output format.

"Here is the JSON:" → probability zero. Can't be generated.

`response_format={"type": "json_object"}` is this. A prompt instruction is not.

---

**Tweet 4 (why this matters for agents):**
Same mechanism, different API call:

`tool_choice="auto"` → model probabilistically decides whether to call the tool

`tool_choice="required"` → constrained decoding, model must emit a tool call

"My agent sometimes skips the tool" is almost always this gap.

---

**Tweet 5 (the real fix):**
Before:
```python
client.chat.completions.create(model=..., messages=[...])
```

After:
```python
client.chat.completions.create(
  model=...,
  messages=[...],
  response_format={"type": "json_object"}
)
```

One parameter. Not three paragraphs in the system prompt.

---

**Tweet 6 (link):**
Understanding this mechanism tells you exactly where the lever is in any pipeline that acts on structured LLM output.

Full explainer (with the Week 11 artifact context):
https://substack.com/profile/496969358-kirubel/note/c-256332814

---
*Published: 2026-05-08 — https://x.com/kirubeltewodro2/status/2053154330803331531*
