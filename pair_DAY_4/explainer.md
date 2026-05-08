# Explainer — Day 4
**Question answered:** In my Week 11 tenacious-bench pipeline (run_sealed_ablation.py + model_card.md), which reported comparisons require paired bootstrap, which can use non-paired alternatives, and what is the minimal patch that keeps my CI/p-value claims statistically honest?
**Written by:** Kirubel Tewodros
**Date:** 2026-05-08 (Friday)

---

## The Setup

You ran a paired bootstrap on Delta A and Delta B in your Week 11 ablation — and that was the right call. But your model_card.md also reports latency deltas and cost claims without the same treatment. With n_pairs=22, one pair flipping changes your observed rate by ~4.5 percentage points. So the question of *when* to pair and *when not to* is not academic — it directly determines whether your "trained beats baseline" claim survives review or collapses under scrutiny.

---

## What Pairing Actually Does

A paired bootstrap resamples the evaluation *unit* — in your case, the (prompt, condition_A_score, condition_B_score) triple — and computes the difference within each resample. This is different from resampling condition A and condition B independently.

The reason this matters: your held-out tasks are not equally hard. Some prompts reliably trip the agent; others are easy for any condition. When you pair, that per-task difficulty gets cancelled out in every resample, because both conditions face the same draw. When you don't pair, task difficulty bleeds into your estimate of the treatment effect — inflating variance and widening your CIs for no good reason.

A concrete illustration with your numbers: suppose 5 of your 22 pairs are systematically hard tasks where both conditions fail. With paired bootstrap, those 5 always fail together, so they don't affect the delta estimate. With unpaired bootstrap, those 5 hard tasks might randomly land in condition A's resample but not B's, making condition A look artificially worse. Your CI gets wider and your p-value rises — not because the treatment effect is weaker, but because you lost the blocking benefit.

---

## The Decision Table for Your Pipeline

| Comparison | Pairing required? | Why |
|---|---|---|
| Delta A: trained accuracy vs baseline accuracy | **Paired — required** | Same 22 held-out pairs scored under both conditions. Task-level difficulty is a real confounder. Unpairing inflates variance. |
| Delta B: trained accuracy vs prompt-only accuracy | **Paired — required** | Same reason: same pairs, same tasks. Both conditions see the same prompts. |
| Latency delta (LoRA vs prompt-only) | **Descriptive only** | Latency is a system property, not a per-pair outcome. There is no meaningful within-pair correlation to exploit. Report mean ± std or a percentile range. No p-value needed. |
| Cost delta | **Descriptive only** | Same as latency — cost per task is determined by the pipeline, not by the prompt content. A CI here overstates inferential precision. |
| Margin-level accuracy (e.g. accuracy on a subset of tiers) | **Paired if same pairs; unpaired if subset changes** | If the subset uses the same held-out pairs evaluated under both conditions, pair. If you're comparing two different subsets (e.g. inquiry tier tasks vs hypothesis tier tasks), these are independent samples — unpaired or descriptive. |

---

## Why Unpaired Would Be Wrong for Delta A/B

The intuition: if you resample condition A's 22 scores and condition B's 22 scores independently, you are asking "what if these were two different experiments?" But they weren't — they were the same 22 tasks scored twice. You have a natural experiment design, and unpaired bootstrap throws away that structure. For small n (22), this matters significantly. At n=200+, paired and unpaired bootstrap converge, and the choice matters less.

---

## The Minimal Patch

Three changes keep your claims honest without rebuilding the stack:

**1. Confirm the resample unit in run_sealed_ablation.py.**
The bootstrap should resample the index `i` from `range(n_pairs)` and compute `delta[i] = score_A[i] - score_B[i]` in each resample — not resample `score_A` and `score_B` independently. If the current code does `np.random.choice(score_A)` and `np.random.choice(score_B)` separately, that is unpaired and needs to change to `np.random.choice(range(n_pairs))`.

**2. Strip CI language from latency and cost in model_card.md.**
Replace "95% CI [x, y]" for latency/cost claims with "mean ± std across held-out tasks" or a percentile range. These are descriptive measurements, not inferential comparisons. Claiming a CI implies a hypothesis test that doesn't apply here.

**3. Add one-line scope notes for margin-level claims.**
For any per-tier or per-category accuracy claim, add a parenthetical: "(n=k pairs, descriptive)" if the subsample is too small for inference, or confirm pairing explicitly if you're comparing the same pairs under two conditions.

---

## Adjacent Concept: Why n=22 Makes Pairing Critical

Power analysis for a paired test at n=22: to detect a true difference of +0.10 at 80% power with α=0.05, you need the within-pair correlation to be reasonably high (r > 0.3). If tasks are truly homogeneous (no difficulty variation), pairing gains you nothing. In practice, evaluation benchmarks have substantial task-level variance — some prompts are just harder — so within-pair correlation is usually high, and pairing gives you a meaningful power boost. At n=22, this is not a minor statistical nicety; it's the difference between a CI that barely excludes zero and one that comfortably does.

---

## One-Line Summary

Pair when the same evaluation units appear in both conditions (Delta A, Delta B — always pair); go descriptive when the metric is a system property with no per-unit pairing structure (latency, cost). The minimal patch is confirming the resample index is shared, not independent.

---

## Sources

- [An Introduction to the Bootstrap](https://www.taylorfrancis.com/books/mono/10.1201/9780429246593/introduction-bootstrap-bradley-efron-robert-tibshirani) (Efron & Tibshirani, 1993) — the canonical reference; Chapter 9 covers paired vs unpaired bootstrap and the variance-reduction argument
- [The Hitchhiker's Guide to Testing Statistical Significance in NLP](https://aclanthology.org/P18-1128/) (Dror et al., ACL 2018) — directly addresses when to pair in NLP evaluation pipelines; the dependency assumption is explained in plain language
- [Accounting for Variance in Machine Learning Benchmarks](https://arxiv.org/abs/2103.03098) (Bouthillier et al., 2021) — shows empirically that small held-out sets (n<50) produce high-variance benchmark rankings unless variance is explicitly modelled

## Tool Used
- **Tool:** Claude Code with direct artifact access (Week 11 memo.md, run_ablation.py, model_card.md)
- **What you ran:** Cross-referenced the ablation script structure and model_card reporting against Hiwot's artifact anchors to classify each comparison type
- **What it showed:** Delta A/B use the same held-out pairs under both conditions — paired bootstrap is structurally required; latency/cost are system properties with no per-pair structure — CI language there overstates precision
