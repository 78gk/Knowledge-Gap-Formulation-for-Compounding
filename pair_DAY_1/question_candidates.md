# Gap Candidates — Day 1
**Topic:** Inference-time mechanics
**Artifact analyzed:** "Building Tenacious Bench: What τ² Bench Can't Measure" (HuggingFace article)
**Generated:** 2026-05-04

Bring all 5 to your morning call with Nurye. Commit to one after interrogating each other's drafts.

---

## Gap Candidate 1 ⭐ RECOMMENDED
**Research value:** HIGH

**The gap:** You don't know whether your LoRA adapter was merged into the base weights before inference or kept separate — and these have fundamentally different latency profiles.

**Where it shows in your artifact:**
> "LoRA adapter (this work): 0.3065" and "real inference on both sides — no simulation, no approximation"

You claim "real inference" as a quality signal but never describe what inference with a LoRA adapter actually means mechanically. `merge_and_unload()` collapses the adapter into the base model weights (zero overhead). Keeping it separate adds two matrix multiplications (A×B) per adapted layer per forward pass. You don't say which you did.

**Why it matters:** If you merged, your inference speed equals the base model — and you can say so defensibly. If you didn't, your "under 2 seconds" number has a hidden overhead you haven't quantified. This changes how you'd describe and optimize the system in a production context.

**Draft question:** When a LoRA adapter trained on q_proj and v_proj is applied to a 0.5B model at inference time, what exactly happens to the attention computation — is the adapter weight added as a separate forward pass through the low-rank matrices, or merged into the frozen weights, and what are the latency implications of each approach on memory-bandwidth-bound hardware like a T4?

---

## Gap Candidate 2
**Research value:** HIGH

**The gap:** You don't know whether your "under 2 seconds per task" evaluator runtime is prefill-dominated or decode-dominated — and therefore can't reason about how to make it faster.

**Where it shows in your artifact:**
> "The scoring evaluator runs in under 2 seconds per task with no GPU required."

You state this as a quality signal (fast, CPU-runnable) but can't derive it. You don't know the token budget per task, the prefill time, the number of output tokens, or which phase bottlenecks it.

**Why it matters:** Prefill and decode have opposite scaling behavior. Prefill scales with input length and parallelizes well. Decode is sequential and memory-bandwidth bound. Knowing which dominates tells you whether to shorten prompts or reduce output tokens to cut cost — and changes your benchmark's cost model entirely.

**Draft question:** For a task with a 300-token input prompt and a 50-token output, on a CPU inference run with a 0.5B model, what fraction of total wall-clock time is spent in the prefill phase vs the decode phase — and what would you change to cut the total from 2 seconds to 0.5 seconds?

---

## Gap Candidate 3
**Research value:** HIGH

**The gap:** You don't know why prefix caching would have reduced your scoring evaluator's cost, even though your eval pipeline is the ideal setup for it.

**Where it shows in your artifact:**
> Your training config (system prompt with threshold rules) + 260 tasks in the benchmark

Your 260 tasks all share the same system prompt specifying the threshold rules. Every evaluation call re-processes this shared prefix from scratch. You never mention prefix caching or KV cache reuse.

**Why it matters:** This is a direct, concrete improvement to your own system. Knowing KV cache + prefix caching mechanics would let you cut your evaluator runtime for the shared prefix to near-zero on the second through 260th call. It also changes how you'd architect any future multi-task evaluation pipeline.

**Draft question:** In a pipeline that runs 260 inference tasks all sharing the same 200-token system prompt, how does KV cache prefix caching work at the token level — what gets stored, when does it get reused, and what is the actual compute saving on a typical LLM inference stack?

---

## Gap Candidate 4
**Research value:** MEDIUM-HIGH

**The gap:** You chose q_proj and v_proj as LoRA target modules but can't explain what this choice does to the attention computation at inference.

**Where it shows in your artifact:**
> "LoRA rank=16, alpha=32, target modules q_proj+v_proj"

You made a specific architectural choice — adapting Q and V but not K, not the feed-forward layers — but the article treats it as config rather than a defended decision.

**Why it matters:** This choice affects which part of the attention mechanism learns to change its behavior. Targeting q_proj changes how the model forms queries (what it looks for). Targeting v_proj changes what it retrieves. Not targeting k_proj means the keys (what it looks at) stay frozen. Understanding this would let you explain and defend your adapter design to an FDE client or reviewer.

**Draft question:** In a transformer attention head, what is the precise role of q_proj vs k_proj vs v_proj in the computation — and when training a LoRA adapter for a behavioral phrasing task, why might adapting q_proj and v_proj be sufficient while leaving k_proj frozen?

---

## Gap Candidate 5
**Research value:** MEDIUM

**The gap:** You don't know whether your 0.5B model on a T4 is compute-bound or memory-bandwidth-bound at inference, which changes every latency optimization decision.

**Where it shows in your artifact:**
> "T4 16bit" and "Actual wall-clock: 34.8 minutes on T4"

You describe your hardware and precision but can't explain why it took 34.8 minutes (not 5, not 120). You don't know the arithmetic intensity of your model at this size on this hardware.

**Why it matters:** If decode is memory-bandwidth-bound (which it almost certainly is for a 0.5B model), then increasing batch size improves throughput but not per-request latency. This changes how you'd deploy this model in production and whether the "T4 16bit" choice was optimal.

**Draft question:** For a 0.5B parameter transformer model running autoregressive decode on a T4 GPU, is the bottleneck compute (FLOPS) or memory bandwidth (moving weights from HBM to SRAM per token), and what does the answer imply for how to optimize inference throughput vs latency?

---

## Triage Summary

| # | Gap | Research Value | Generalizable? | Resolvable in 600-1000 words? |
|---|-----|---------------|---------------|-------------------------------|
| 1 | LoRA merged vs separate at inference | HIGH | Yes — any LoRA deployment | Yes |
| 2 | Prefill vs decode split | HIGH | Yes — any LLM inference | Yes |
| 3 | KV cache + prefix caching | HIGH | Yes — any multi-task eval pipeline | Yes |
| 4 | q_proj vs v_proj targeting | MEDIUM-HIGH | Moderate — LoRA practitioners | Yes |
| 5 | Compute-bound vs memory-bandwidth-bound | MEDIUM | Yes — any inference optimization | Yes |

**Suggested starting point for morning call:** Gap 1 (LoRA at inference) or Gap 2 (prefill/decode split) — both are directly grounded in numbers you stated in your article that you can't fully defend.
