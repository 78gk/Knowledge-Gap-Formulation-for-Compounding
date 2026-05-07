# Tweet Thread — Day 3
**Platform:** Twitter/X
**Topic:** Training and post-training mechanics — LoRA rank
**Date:** 2026-05-07 (Thursday)

---

**Tweet 1 (hook):**
I trained a LoRA adapter that scored well on every benchmark — then watched it fail consistently on one specific input type in production.

The reason came down to one number I had never questioned: the rank.

Here's what rank actually does, and why benchmarks can't tell you when it's wrong. 🧵

---

**Tweet 2 (mechanism):**
LoRA doesn't update a model's full weight matrices. Instead it learns a small shortcut:

ΔW = B · A

where r (the rank) is the bottleneck dimension.

With rank=16, the adapter can only move the model in 16 independent directions — no matter how much data you throw at it.

That ceiling is set by architecture, not training.

---

**Tweet 3 (why benchmarks miss it):**
Benchmarks measure average performance across a distribution.

Most fine-tuning tasks are genuinely low-dimensional — a rank-16 adapter is more than enough to get the broad directions right.

What benchmarks don't test: the thin slices at the edge of two categories.

That's exactly where a rank-limited adapter runs out of room.

---

**Tweet 4 (concrete example):**
In my Week 11 project, the adapter learned "high confidence → assertive phrasing" and "low confidence → hedge" perfectly.

But at confidence=0.55 — right on the boundary between two tiers — it consistently picked the wrong one.

Not a data problem. The rank-16 subspace simply didn't have a direction for that fine-grained edge.

---

**Tweet 5 (what to do about it):**
Three knobs control what a LoRA adapter can learn:

• Rank — sets the expressiveness ceiling. For boundary-sensitive tasks, try 64–128.
• Alpha — scales the update magnitude, not the subspace. α=2r is convention, not law.
• Target modules — q/v only is safe default; add k_proj if the task needs fine-grained attention matching.

Benchmark first. Then check the edges.

---

**Tweet 6 (link):**
Full explainer — the mechanism, why distribution shift makes it worse, and what I'd change in my own config:

INSERT_BLOG_URL

#MachineLearning #LoRA #Finetuning #LLM

---
*Each tweet must stand alone. Replace INSERT_BLOG_URL before publishing.*
