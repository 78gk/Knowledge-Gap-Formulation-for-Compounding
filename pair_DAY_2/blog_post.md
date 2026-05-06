# "Return JSON Only" Is Not a Command — It's a Wish

*Day 2 of my AI engineering knowledge gap series. Topic: agent and tool-use internals.*

---

My Week 11 pipeline had a judge. An LLM judge — DeepSeek V3 — that was supposed to evaluate each generated benchmark task and return a structured score with six fields: coherence, verifiability, rubric clarity, a mean score, a pass/fail, and notes.

Most of the time, it did exactly that.

Occasionally, it returned something like: *"Here is my evaluation of the task: { 'coherence': 3... }"*

That one leading sentence broke everything. `json.loads()` raised an exception. The task was dropped silently. My dataset construction pipeline had a hole in it and I didn't know.

My fix: I added "respond only in JSON, no preamble" to the system prompt.

It seemed to work. I shipped it. I moved on.

Here is what I didn't know: I had not fixed anything. I had made a fragile thing slightly less likely to break.

---

## Two Mechanisms

When you tell an LLM to return structured output, there are two completely different things that can happen.

**The first is instruction following.** The model has been trained — through millions of examples and reinforcement learning — to comply with user instructions. When it sees "return JSON only," it has learned that in this context, the next tokens should be JSON. So it assigns high probability to JSON tokens and low probability to everything else.

High probability is not certainty. It's not even a guarantee. It's a learned preference. If the context is unusual, if the conversation is long, if the input is at the edge of the training distribution — the model can still deviate. The system prompt instruction shifted the distribution. It did not eliminate alternatives.

**The second is constrained decoding.** This is completely different. At every single token generation step, the model produces a probability distribution over its vocabulary — 32,000 to 100,000 tokens. In constrained decoding, a grammar is applied at each step to mask out every token that would make the output invalid according to the target format.

If the model is generating a JSON object and the only valid next tokens are a key name, a number, `true`, `false`, or `}`, then every other token in the vocabulary receives a probability of exactly zero. The model cannot generate "Here is my evaluation" — those tokens do not exist in its output space at that moment. Not unlikely. Impossible.

---

## What Actually Changes When You Set `response_format`

`response_format={"type": "json_object"}` in the OpenAI or OpenRouter API is not a smarter version of a prompt instruction. It is a different mechanism entirely. It activates grammar-constrained decoding at the inference level. The JSON grammar is loaded. Invalid tokens are masked at each step. The model has no path to freeform output.

My pipeline, with only a prompt instruction, was running Mechanism 1 on every judge call. Over 260 tasks, that small non-zero failure probability compounds. I got lucky. A single parameter change makes it structural.

```python
# What I had — instruction following
response = client.chat.completions.create(
    model="deepseek/deepseek-chat-v3-0324",
    messages=[{"role": "system", "content": judge_system_prompt},
               {"role": "user", "content": judge_prompt}],
    temperature=0.2
)

# What I needed — constrained decoding
response = client.chat.completions.create(
    model="deepseek/deepseek-chat-v3-0324",
    messages=[{"role": "system", "content": judge_system_prompt},
               {"role": "user", "content": judge_prompt}],
    temperature=0.2,
    response_format={"type": "json_object"}
)
```

---

## Why This Matters for Every Agent You Build

This is not a niche JSON parsing issue. This is the exact mechanism behind unreliable tool-calling in LLM agents.

When an agent uses `tool_choice="auto"`, the model probabilistically decides whether to call a tool or answer directly. It will usually call the tool when the context suggests it should. But it can skip it — and when it does, you get a hallucinated answer instead of a tool-grounded one, with no error, no signal, no way to catch it downstream.

`tool_choice="required"` is constrained decoding. The model must emit a tool call object. It is incapable of producing a plain text response instead.

Every agent behavior that looks like "the model sometimes doesn't call the tool" is almost always this: the system was relying on instruction following when it needed constrained decoding. The fix is the same as my JSON fix — an API parameter, not a longer system prompt.

---

## One Nuance Worth Adding

After the evening call with my partner, one thing became clearer: `json_object` mode guarantees valid JSON syntax, not correct field names. If you need the model to return exactly the fields your code expects — no more, no fewer — you need `response_format={"type": "json_schema", "json_schema": {...}}` with an explicit schema. That's a second level of enforcement that `json_object` alone doesn't give you.

---

The real lesson from Day 2: when your pipeline relies on an LLM to do something reliably, "reliably" has a mechanism. Find the mechanism. Then set the right parameter. The system prompt is not the mechanism.

---

*Sources: Willard & Louf (2023), "Efficient Guided Generation for Large Language Models" — the grammar-constrained decoding paper. OpenAI Structured Outputs documentation.*
