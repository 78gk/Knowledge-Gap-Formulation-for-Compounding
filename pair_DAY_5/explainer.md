# Explainer — Day 5
**Question answered:** I trained a LoRA SFT adapter on my Week 11 agent and it showed calibration failures on edge inputs. DPO and ORPO are listed as post-SFT alignment steps. What does DPO actually optimize — how is its training objective different from SFT — and would it have addressed calibration failure specifically, or is it solving a different problem?
**Written by:** Kirubel Tewodros
**Date:** 2026-05-09 (Saturday)

---

## The Gap

SFT (supervised fine-tuning) teaches a model to imitate demonstrations: given this input, produce this output. If you have labeled examples of good behavior, SFT pushes the model's probability mass toward those outputs. That is what Bereket's LoRA adapter did — it learned to produce the phrasing-tier outputs in the training set.

But SFT has a structural limitation: it tells the model what to do, not what to avoid. If two outputs look equally plausible to the model — say, the "inquiry" tier and the "hypothesis" tier at a confidence boundary — SFT has no mechanism to create a preference between them. It can only reward the demonstrated output; it cannot penalize the alternative. DPO (Direct Preference Optimization) is designed exactly for this case.

---

## What SFT Actually Optimizes

SFT maximizes the log-likelihood of the demonstrated output given the input:

```
L_SFT = -log p_θ(y_chosen | x)
```

This is a maximum likelihood objective. The model is pushed to assign high probability to `y_chosen`. What happens to `y_rejected` — the wrong phrasing tier, the overconfident assertion — is not specified. If `y_rejected` is in a high-probability region, SFT leaves it there.

For easy cases (high confidence → assertive, very low confidence → abstain), this is fine. The correct output is so much more likely that SFT's push in the right direction is enough. But at the boundary — where both outputs have similar prior probability — SFT provides no gradient signal to separate them.

---

## What DPO Optimizes

DPO (Rafailov et al., 2023) takes a different starting point. Instead of maximizing likelihood of demonstrations, it directly optimizes a preference: given an input `x`, make `y_chosen` more likely than `y_rejected`, relative to a reference model.

The DPO training objective:

```
L_DPO = -log σ(β · (log p_θ(y_chosen|x)/p_ref(y_chosen|x) - log p_θ(y_rejected|x)/p_ref(y_rejected|x)))
```

Breaking this apart:
- `p_θ` is the model being trained; `p_ref` is the frozen reference model (typically the SFT checkpoint)
- The inner expression measures how much `p_θ` prefers `y_chosen` over `y_rejected`, relative to what the reference model would predict
- `β` controls how far the trained model can move from the reference — low β = conservative, high β = aggressive update
- The loss increases when the model assigns similar or reversed preference between the two outputs

The key insight: DPO's gradient signal is zero when the model already prefers `y_chosen` strongly. It only fires when the model is uncertain — exactly at the boundary cases that SFT misses.

---

## Why This Means DPO and SFT Solve Different Problems

| | SFT | DPO |
|---|---|---|
| **Training signal** | "Produce this output" | "Prefer this over that" |
| **Data needed** | Demonstrations (input → output) | Preferences (input → chosen vs. rejected) |
| **Gradient at confident predictions** | Always nonzero — every example contributes | Near-zero — model already correct |
| **Gradient at uncertain predictions** | Same as confident — no special treatment | High — boundary cases get strong signal |
| **What breaks silently** | Boundary cases with similar likelihoods | Reward hacking if preferences are inconsistent |

For Bereket's calibration failure: SFT could not fix it because the training data didn't provide enough boundary examples to push the model past the ambiguity. DPO could fix it if paired preference data is available — for each boundary input, label which output is correct and which is wrong. That labeled contrast is the training signal SFT never had.

---

## ORPO: One Step Further

ORPO (Hong et al., 2024) integrates SFT and preference alignment into a single training pass. Instead of SFT first, then DPO on the SFT checkpoint, ORPO adds an odds-ratio penalty directly to the SFT objective:

```
L_ORPO = L_SFT - λ · log(odds_ratio(y_chosen, y_rejected))
```

The odds ratio penalizes the model for assigning similar probability to chosen and rejected outputs during the same forward pass that also maximizes likelihood of chosen. This eliminates the two-stage training pipeline and avoids the reference model overhead.

For production use, ORPO is appealing because: (1) you need half the GPU memory (no reference model copy), (2) the single-pass training is faster, (3) the implicit reference is the model's own predictions at each step. The tradeoff: the `β` parameter in DPO is more interpretable and gives more explicit control over how far the model can shift from its pre-alignment behavior.

---

## The Direct Answer to the Question

Would DPO have fixed Bereket's calibration failure? **Only if the failure is a preference problem, not a representation problem.**

From Day 3 research on LoRA rank: if the boundary failure is a representational failure (the rank-16 subspace cannot express the fine-grained decision surface), then DPO will also fail — because the model cannot represent the distinction regardless of what gradient signal it receives. DPO cannot fix what ΔW = B·A cannot express.

If, however, the boundary failure is a preference problem (the model has the representational capacity but SFT training didn't give it a signal to distinguish the two outputs at the margin), then DPO with paired boundary examples would be exactly the right tool.

The practical diagnostic: try rank=64 with k_proj added (the Day 3 recommendation) first, because it addresses the representational limit. If boundary failures persist after that change, then preference data + DPO is the next lever.

---

## Sources

- [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/abs/2305.18290) (Rafailov et al., NeurIPS 2023) — derives the DPO objective from first principles; shows it is mathematically equivalent to RLHF with a specific reward parameterization but requires no separate reward model
- [ORPO: Monolithic Preference Optimization Without Reference Model](https://arxiv.org/abs/2403.07691) (Hong et al., 2024) — introduces the odds-ratio penalty that eliminates the reference model; directly comparable to DPO on Llama and Phi benchmarks
- [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155) (Ouyang et al., 2022) — original InstructGPT paper establishing the RLHF framework that DPO replaces; context for understanding what DPO simplifies

## Tool Used
- **Tool:** Claude Code with direct Week 11 artifact access
- **What you ran:** Cross-referenced Day 3 grounding commit (model_card.md, rank-16 representational failure) against the DPO objective to assess whether the calibration failure is representational or preferential
- **What it showed:** The Day 3 analysis identifies a representational failure as the primary cause. DPO is a second-order fix: valuable if rank is increased first and boundary failures persist, but not a substitute for addressing the subspace constraint.
