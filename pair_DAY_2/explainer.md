# Explainer — Day 2
**Question answered:** What is the exact mechanism by which an LLM emits a tool call when the API includes `tools=[...]`? Special token or post-hoc JSON parse?
**Written by:** Kirubel Tewodros (for Natnael Alemseged)
**Date:** 2026-05-06 (Wednesday)

---

## The Gap

Natnael's Week 10 Conversion Engine describes itself as an "agent that uses tools" — Cal.com booking, HubSpot CRM updates. But `openrouter_llm.py` (lines 45–70) sends vanilla chat completions with no `tools` parameter. The routing decision — which tool to call — is made by Python keyword matching in `lead_orchestrator.py` line 371 (`_booking_intent`), not by the model. The model is only called for argument generation, not tool selection.

Then τ²-Bench evaluation traces showed `finish_reason: "tool_calls"`. That's a signal the model emitted a tool call. But Natnael's production code never passes `tools=[...]`. The question: what mechanism produces that response, and how is it different from what his system does?

---

## What Actually Happens When You Pass `tools=[...]`

When a request includes `tools=[...]`, the provider does three things before the model ever generates a token:

**Step 1 — Schema injection.** The tools array (function names, descriptions, parameter schemas) is serialized and injected into the model's context. Depending on the provider and model, this happens as a special system message, a formatted block appended to the system prompt, or — for models with purpose-built tool-calling fine-tuning — as a structured input that maps to special tokens in the tokenizer.

**Step 2 — Model generates a structured response.** The model has been fine-tuned to recognize the tool schema in its context and, when the user request matches a tool, emit a response in a specific format. For OpenAI-compatible APIs, that format is:

```json
{
  "tool_calls": [{
    "id": "call_abc123",
    "type": "function",
    "function": {
      "name": "book_meeting",
      "arguments": "{\"lead_email\": \"alex@corp.com\", \"slot\": \"2026-05-07T14:00\"}"
    }
  }]
}
```

**Step 3 — Provider parses and returns structured data.** The provider detects this output pattern and returns it as the `tool_calls` field in the response object, setting `finish_reason: "tool_calls"`.

---

## Special Token or Post-Hoc JSON Parse?

The answer is: **both exist, depending on the model and serving infrastructure.**

**Special tokens** are used by many open-source models. Qwen3 uses `<tool_call>` and `</tool_call>` as dedicated tokens in its tokenizer. Llama 3 uses `<|python_tag|>`. When the model generates one of these tokens, the provider (or local inference engine) treats it as a trigger and extracts the content that follows. The token is never shown in the text output — it's intercepted.

**Post-hoc JSON parsing** is what happens when a model generates JSON-formatted output that matches the expected tool call structure, and the provider parses it after the fact. This is less robust — it depends on the model consistently outputting valid JSON in the right shape — but it works for well-fine-tuned models and is what most OpenRouter-normalized tool calling does for models without dedicated tool tokens.

**For Natnael's stack (OpenRouter):** OpenRouter normalizes tool calling across models. For models with native tool support, it uses the model's special tokens or structured output path. For models without it, it injects the schema as a system prompt and parses the output post-hoc. The `finish_reason: "tool_calls"` in the τ²-Bench traces means the evaluation framework was using the former path — with a model that genuinely emits tool call tokens.

---

## How This Differs From `_safe_parse_json`

Natnael's current system uses `_safe_parse_json` to extract structured arguments from LLM output after the routing decision is already made by Python. Here's the comparison:

| | Native `tools=[...]` | `_safe_parse_json` |
|---|---|---|
| **Who decides which tool** | The model (sees schema, chooses) | Python substring matching |
| **Parse reliability** | Provider-built parser, purpose-trained | Generic JSON fallback, can fail on edge cases |
| **Refuse/abstain** | `finish_reason: "stop"` = model chose to answer directly | No signal — parse failure looks identical to "chose not to call" |
| **Latency** | +tokens for schema injection in prefill | No schema overhead |
| **What breaks silently** | Nothing — failure is explicit | Parse failure can corrupt downstream state |

The critical difference is the **refuse/abstain signal**. With native tool calling, the model can communicate "I understood the request but a tool call isn't appropriate here" via `finish_reason: "stop"`. With `_safe_parse_json`, there is no way to distinguish a model that chose not to call a tool from a model that tried and formatted wrong.

---

## What This Means for Natnael's Architecture

His Week 10 system is not misengineered — it's a deliberate **orchestrated workflow**: Python makes routing decisions, the model fills in arguments. This is often more reliable in production than pure model-driven tool selection, because the routing logic is deterministic.

But the portfolio language "agent that uses tools" misrepresents it. An agent with model-visible tool schema means the model sees the tool definitions and decides whether and which to call. Natnael's system doesn't do that. The honest description is: "orchestrated workflow with LLM-assisted argument generation." That's a valid and often correct architectural choice — it just has a different name.

---

## Sources
- [Toolformer: Language Models Can Teach Themselves to Use Tools — Schick et al., 2023](https://arxiv.org/abs/2302.04761) — original paper establishing the paradigm of models learning to emit tool calls as text sequences during generation
- [OpenAI Function Calling Documentation](https://platform.openai.com/docs/guides/function-calling) — documents the request/response contract for `tools=[...]` including `finish_reason` semantics and the `tool_calls` response structure
