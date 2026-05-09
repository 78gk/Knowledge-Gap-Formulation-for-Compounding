# Canonical Reading List — Week 12 Contribution
**Author:** Kirubel Tewodros
**For:** 10Academy FDE Cohort Canon

This is my annotated contribution to the cohort's shared knowledge base. Each entry is something I read this week, verified against the primary source, and judge worth another Forward-Deployed Engineer's time.

---

## Papers

| Paper | Authors | Year | Why It Matters for FDE Work |
|-------|---------|------|----------------------------|
| [Attention Is All You Need](https://arxiv.org/abs/1706.03762) | Vaswani et al. | 2017 | Establishes the KV cache: key-value matrices computed during prefill are stored and reused at decode. Without reading this, "why is decode slow but prefill fast?" has no answer. |
| [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685) | Hu et al. | 2021 | Defines rank decomposition ΔW=B·A and the α/r scaling convention. Read this before choosing any LoRA config — rank is a hard expressiveness ceiling, not a tuning knob. |
| [Intrinsic Dimensionality Explains Fine-Tuning Effectiveness](https://arxiv.org/abs/2012.13255) | Aghajanyan et al. | 2021 | Shows why low-rank adapters work on benchmarks: most fine-tuning tasks are intrinsically low-dimensional. Also explains why benchmarks miss boundary failures. |
| [Direct Preference Optimization](https://arxiv.org/abs/2305.18290) | Rafailov et al. | 2023 | The cleanest explanation of why SFT alone is not sufficient for aligned behavior. DPO's gradient fires at uncertain boundaries — exactly where SFT's gradient is silent. |
| [Efficient Guided Generation for LLMs](https://arxiv.org/abs/2307.09702) | Willard & Louf | 2023 | The grammar-constrained decoding algorithm behind `response_format`. Every FDE who parses LLM output in a pipeline needs to understand the difference between token masking and instruction-following. |
| [Fast Inference via Speculative Decoding](https://arxiv.org/abs/2211.17192) | Leviathan et al. | 2023 | Proves speculative decoding preserves output distribution exactly while reducing sequential decode steps. Essential for understanding the inference optimization design space — and knowing when it doesn't apply. |
| [The Hitchhiker's Guide to Statistical Significance in NLP](https://aclanthology.org/P18-1128/) | Dror et al. | 2018 | Direct practical guidance on bootstrap paired vs unpaired, when p-values are appropriate, and what CI width communicates. Should be standard reading before publishing any benchmark results. |
| [LoRA Learns Less and Forgets Less](https://arxiv.org/abs/2405.09673) | Biderman et al. | 2024 | Empirical evidence that rank-limited adapters under-perform on distribution-shifted evaluation while preserving in-distribution scores. The missing paper for any LoRA deployment recommendation. |
| [ORPO: Monolithic Preference Optimization Without Reference Model](https://arxiv.org/abs/2403.07691) | Hong et al. | 2024 | Single-pass SFT + preference alignment. Half the GPU memory of DPO. The production-pragmatic alternative when two-stage training is expensive. |
| [Accounting for Variance in ML Benchmarks](https://arxiv.org/abs/2103.03098) | Bouthillier et al. | 2021 | Shows empirically that small held-out sets (n<50) produce high-variance benchmark rankings. Motivates why CI width matters more than point estimates for deployment decisions. |

---

## Tools

| Tool | What It Does | How I Used It | When to Reach for It |
|------|-------------|---------------|----------------------|
| [Anthropic Python SDK](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) | Exposes `cache_read_input_tokens` in response usage — the direct measurement of prefix cache hits | Called the same prompt assembly with broken and fixed structure across 3 consecutive requests; measured latency and cache tokens per call | Whenever validating whether a prompt structure change is actually hitting the prefix cache |
| [OpenRouter API](https://openrouter.ai) | Normalizes tool calling and structured output across multiple model providers via OpenAI-compatible interface | Ran 20 judge-prompt edge cases with and without `response_format` to measure format failure rate | Multi-model pipelines where you want to test constrained decoding behavior across providers without changing your SDK |
| [Paired bootstrap resampling](https://aclanthology.org/P18-1128/) (pattern, not a library) | Resamples paired (input, score_A, score_B) triples rather than independent score arrays | Verified the index resampling structure in run_sealed_ablation.py; classified each comparison in the pipeline as paired-required or descriptive-only | Whenever comparing two model conditions evaluated on the same held-out tasks |

---

## Engineering Patterns

**Stable-prefix isolation**
What it is: Assembling LLM prompts with all stable content (system instructions, policy rules, tool definitions, few-shot examples) as an unbroken constant string, with volatile per-request data appended strictly after.
When it applies: Any agent or pipeline with large stable system prompts and per-request variable data. Worth implementing even at low request volume — the cost of getting it wrong is full prefill cost on every call.
Common mistake: Embedding volatile fields inside the stable block using f-strings. The provider hashes the prefix up to the first volatile token — any volatile content inside the stable block makes the prefix unique per call.
Evidence: Anthropic prompt caching docs; measured 85% latency reduction in Day 1 research on identical prompt content, different structure.

**Response-format-first output engineering**
What it is: Defaulting to API-level output format parameters (`response_format`, `tool_choice`) rather than relying solely on prompt instructions for structured output.
When it applies: Any pipeline that parses LLM output downstream — JSON parsing, tool routing, score extraction. If a parse failure would be silent or corrupt downstream state, instruction-following is insufficient.
Common mistake: Treating "return JSON only" in a system prompt as equivalent to `response_format={"type": "json_object"}`. They operate through different mechanisms; one is probabilistic, the other is structural.
Evidence: Willard & Louf (2023); measured 3/10 format failures on edge cases with instruction-only vs 0/10 with constrained decoding in Day 2 research.

**Rank-before-preference diagnostic**
What it is: Before applying preference optimization (DPO, ORPO) to fix model behavior failures, diagnosing whether the failure is representational (the adapter cannot express the correction) or preferential (the adapter can express it but was not given a preference signal).
When it applies: Any fine-tuned model showing failures on boundary or edge-case inputs that passed benchmarks. Also applies when planning post-SFT alignment steps.
Common mistake: Adding preference data and running DPO on a model whose failure is caused by rank constraints. DPO cannot add representational capacity — it can only reshape the distribution within the existing subspace.
Evidence: Aghajanyan et al. (2021); Biderman et al. (2024); Day 3 and Day 5 research.

---

## What I'd Add to the Cohort's Next Sprint

Two questions came up this week that weren't on the topic slate but would close important gaps for the cohort:

**Multi-block prefix caching with `cache_control` headers.** Day 1 covered single-block prefix caching, but many production agents have multiple stable sections — system prompt, tool definitions, few-shot examples — each eligible for independent cache breakpoints. The question of how to set and validate multi-block caching patterns, and how much additional complexity is justified by the incremental savings, was deferred as follow-on in Day 1. For any FDE building multi-tool agents, this is the next optimization to understand.

**Statistical power analysis for evaluation design.** Day 4 closed the gap on what CI width communicates after the fact. What the cohort doesn't yet know is how to design evaluation sets with a target sample size — computing what n is required before running to achieve a given CI width or power at a specific effect size. This would change how we design held-out sets rather than just interpreting the ones we have. For any FDE who will commission or design benchmarks (rather than just run them), this is the upstream skill.
