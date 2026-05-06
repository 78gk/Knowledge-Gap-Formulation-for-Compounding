# Gap Closure Sign-off — Day 2
**Asker:** Kirubel Tewodros
**Explainer written by:** Kirubel Tewodros (self-research, critiqued by Natnael Alemseged)
**Date:** 2026-05-06 (Wednesday)

## Gap Closure Judgment

[x] Closed

## What I Understand Now That I Didn't Before

I now understand that "respond only in JSON" in a system prompt and `response_format={"type": "json_object"}` in an API call are not the same thing — they operate through completely different mechanisms. The prompt instruction shifts the probability distribution toward JSON tokens via learned instruction-following behavior; the API parameter applies grammar-based token masking at each decoding step, making non-JSON output physically impossible. My Week 11 judge pipeline was relying on the former and was fragile as a result. The same distinction explains unreliable tool-calling in agents: `tool_choice="auto"` is probabilistic, `tool_choice="required"` is constrained. The real lever in both cases is an API parameter, not a prompt instruction — and the evening call added one more layer: `json_object` enforces valid JSON syntax only; `json_schema` mode with an explicit schema is needed for field-level enforcement.

## What Remains Open

One sub-question: does constrained decoding carry a measurable latency cost at scale? Grammar checking at each token step adds overhead — how significant is this over 260 judge calls? Not blocking for this gap closure, but relevant for the Week 11 pipeline's cost profile.
