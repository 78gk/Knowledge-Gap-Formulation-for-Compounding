# Tweet Thread — Day 5
**Platform:** Twitter/X
**Topic:** Training and post-training — DPO vs SFT
**Date:** 2026-05-09 (Saturday)

---

**Tweet 1 (hook):**
My LoRA adapter improved average benchmark scores but kept failing at the exact confidence boundary where it mattered most.

I thought I needed more training data. I was wrong about the mechanism.

Here's what SFT can't do — and what DPO was designed for. 🧵

---

**Tweet 2 (what SFT actually optimizes):**
SFT maximizes the likelihood of demonstrations:

"Given this input, produce this output."

It tells the model what to do. It has no mechanism to teach the model what NOT to do.

If two outputs are equally plausible, SFT has no gradient signal to distinguish them. The boundary ambiguity survives training unchanged.

---

**Tweet 3 (what DPO adds):**
DPO (Rafailov et al., 2023) takes a paired preference as input:

"Given this input, choose THIS over THAT."

Its loss fires strongest when the model is uncertain — right at the boundary between outputs.

Where SFT goes silent, DPO has maximum gradient. It's designed for exactly the cases SFT misses.

---

**Tweet 4 (the diagnostic split):**
Before reaching for DPO, ask: is this a preference problem or a representation problem?

If your model CAN express the distinction but SFT didn't give a signal → DPO fixes it.

If your model CANNOT express the distinction (rank too low, modules frozen) → DPO fails too.

Fix the representation ceiling first. Then apply preference alignment.

---

**Tweet 5 (ORPO for production):**
ORPO (Hong et al., 2024) integrates SFT + preference alignment in one pass:

No separate reward model. No two-stage pipeline. Half the GPU memory.

Tradeoff: less explicit control over how far the model shifts from its prior behavior.

For fine-tuned adapters at production scale — ORPO is worth knowing.

---

**Tweet 6 (link):**
Full explainer — DPO objective, why it fires at boundaries not broad cases, how it connects to LoRA rank constraints, and the ORPO alternative:

INSERT_BLOG_URL

#MachineLearning #DPO #RLHF #FineTuning #LLM

---
*Each tweet must stand alone. Replace INSERT_BLOG_URL before publishing.*
