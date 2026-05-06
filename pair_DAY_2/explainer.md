# Explainer — Day 2
**Question answered:** Does telling a model to return JSON actually force it to, or just make JSON more likely? What is the real mechanism?
**Written by:** Kirubel Tewodros
**Date:** 2026-05-06 (Wednesday)

---

## The Bug That Started This

In my Week 11 benchmark pipeline, a DeepSeek V3 judge was supposed to return a structured JSON score with six fields: `coherence`, `verifiability`, `rubric_clarity`, `mean`, `pass`, and `notes`. Most of the time it did. Occasionally it returned something like:

> "Here is my evaluation of the task: { 'coherence': 3, ... }"

That leading sentence broke `json.loads()`. The exception wasn't caught. The task was silently dropped from dataset construction. My fix: add "respond only in JSON" to the system prompt. It seemed to work. I moved on.

What I didn't know: the fix was a patch over a probability, not a solution to a mechanism. Those are completely different things.

---

## Two Mechanisms, One Surface

When you ask an LLM to return structured output, two fundamentally different things can happen under the hood. They look the same from the outside — the model returns JSON — but they have completely different failure profiles.

**Mechanism 1: Instruction Following**

The model has been trained, via supervised fine-tuning and RLHF, to comply with user instructions. When it sees "return JSON only," it has learned that the expected output in this context is JSON. So it assigns high probability to JSON tokens and low probability to tokens like "Here is my evaluation."

High probability is not 1.0. Given an unusual input, a long conversation history, or a context that pushes against the instruction, the model can still deviate. The instruction shifts the probability distribution — it does not eliminate alternatives. This is what a system prompt instruction does. It is soft enforcement.

**Mechanism 2: Constrained Decoding**

At every token generation step, the model produces a probability distribution over its full vocabulary — typically 32,000 to 100,000 tokens. In constrained decoding, a grammar or schema is applied at each step to mask out any token that would make the output invalid according to the target format.

If the model is generating a JSON object and the only valid next characters are a key name, a number, `true`, `false`, or `}`, then every other token in the vocabulary gets a probability of exactly zero. The model is physically incapable of generating "Here is my evaluation" — those tokens do not exist in the output space at that step.

This is hard enforcement. The model cannot deviate. It is not that deviating is unlikely — it is that deviating is impossible.

---

## What `response_format` Actually Does

In the OpenAI and OpenRouter APIs, `response_format={"type": "json_object"}` activates constrained decoding. It is not a prompt instruction. It is a parameter that changes how token sampling works at inference time.

When this parameter is set, a JSON grammar is loaded, invalid tokens are masked at each decoding step, and output is guaranteed to be valid JSON. Without it — even with a system prompt instruction — you are relying on Mechanism 1. Your output is probably JSON. Not definitely.

The practical consequence: my Week 11 judge pipeline had a non-zero probability of returning freeform text on every single call. Over 260 tasks, that probability compounds. I got lucky that the failure rate was low enough not to corrupt the dataset materially. But it was fragile by design.

---

## The Same Physics in Tool Calling

This distinction is exactly what makes LLM agents unreliable when not designed carefully.

When an agent decides whether to call a tool or answer directly, the same two mechanisms apply. `tool_choice="auto"` means the model probabilistically decides — it will usually call the tool when the context suggests it should, but it can skip it. `tool_choice="required"` means constrained decoding: the model must emit a tool call object and is incapable of generating a plain text response instead.

Every unreliable agent behavior that looks like "the model sometimes doesn't call the tool" is almost always this gap: the system was relying on instruction following when it needed constrained decoding.

---

## The Fix

```python
# Before — instruction following (fragile)
response = client.chat.completions.create(
    model="deepseek/deepseek-chat-v3-0324",
    messages=[{"role": "user", "content": judge_prompt}]
)

# After — constrained decoding (robust)
response = client.chat.completions.create(
    model="deepseek/deepseek-chat-v3-0324",
    messages=[{"role": "user", "content": judge_prompt}],
    response_format={"type": "json_object"}
)
```

One parameter. The system prompt instruction about JSON format is now redundant — but harmless.

---

## Sources
- [Efficient Guided Generation for Large Language Models — Willard & Louf, 2023](https://arxiv.org/abs/2307.09702) — canonical paper on grammar-constrained decoding; explains the token masking mechanism
- [OpenAI Structured Outputs Guide](https://platform.openai.com/docs/guides/structured-outputs) — documents how `response_format` activates schema enforcement vs. instruction-only JSON mode

## Tool Used
- **Tool:** OpenRouter API (direct call comparison)
- **What you ran:** Called the same judge prompt with and without `response_format={"type": "json_object"}` on an edge-case input designed to elicit preamble text
- **What it showed:** Without `response_format`, the model prefixed the JSON with a sentence on ~1 in 8 edge-case calls. With `response_format`, zero deviations across 20 calls.
