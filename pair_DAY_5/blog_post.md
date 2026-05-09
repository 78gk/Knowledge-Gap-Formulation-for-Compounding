# What DPO Actually Optimizes — And Why It's Not a Substitute for Fixing Your Adapter's Rank

*Day 5 of my AI engineering knowledge gap series. Topic: training and post-training mechanics — DPO and ORPO.*

---

I trained a LoRA adapter on a sales agent. It improved aggregate benchmark scores. It still failed at the exact confidence boundary where the model had to choose between two phrasing tiers. I had two hypotheses for the next training run: increase rank (Day 3 research said the boundary failure was representational), or add preference data and train with DPO.

I had read about DPO in passing without understanding what it actually optimizes. Before reaching for it, I needed to answer one question: was the boundary failure something DPO could fix, or was it something DPO would slide right past?

---

## What SFT Actually Optimizes

SFT (supervised fine-tuning) maximizes the log-likelihood of the demonstrated output given the input:

```
L_SFT = -log p_θ(y_chosen | x)
```

This is a maximum likelihood objective. The model is pushed to assign high probability to `y_chosen`. What happens to `y_rejected` — the wrong phrasing tier, the overconfident assertion — is not specified. If `y_rejected` is in a high-probability region, SFT leaves it there.

For easy cases (high confidence → assertive, very low confidence → abstain), this is fine. The correct output is so much more likely that SFT's push in the right direction is enough. But at the boundary — where both outputs have similar prior probability — SFT provides no gradient signal to separate them. SFT tells the model what to do; it has no mechanism to teach the model what *not* to do.

---

## What DPO Optimizes

DPO (Rafailov et al., 2023) takes a different starting point. Instead of maximizing the likelihood of demonstrations, it directly optimizes a preference: given an input `x`, make `y_chosen` more likely than `y_rejected`, relative to a reference model.

The DPO training objective:

```
L_DPO = -log σ(β · (log p_θ(y_chosen|x)/p_ref(y_chosen|x)
                  - log p_θ(y_rejected|x)/p_ref(y_rejected|x)))
```

Breaking this apart:
- `p_θ` is the model being trained; `p_ref` is the frozen reference model (typically the SFT checkpoint)
- The inner expression measures how much `p_θ` prefers `y_chosen` over `y_rejected`, relative to the reference model
- `β` controls how far the trained model can move from the reference — low β = conservative, high β = aggressive
- The loss is high when the model assigns similar or reversed preference; low when the model strongly prefers `y_chosen`

The key property: DPO's gradient is near-zero when the model already strongly prefers `y_chosen`. It only fires when the model is uncertain — exactly at the boundary cases that SFT misses.

Crucially, DPO is a **post-SFT step**, not a replacement for SFT. The reference model in the objective is the SFT checkpoint. Starting DPO directly from the base model risks reward hacking on early preference data.

---

## SFT vs DPO Side by Side

| | SFT | DPO |
|---|---|---|
| **Training signal** | "Produce this output" | "Prefer this over that" |
| **Data needed** | Demonstrations (input → output) | Preferences (input → chosen vs rejected) |
| **Gradient at confident predictions** | Always nonzero — every example contributes | Near-zero — model already correct |
| **Gradient at uncertain predictions** | Same as confident — no special treatment | High — boundary cases get strong signal |
| **What it cannot fix** | Boundary cases with similar likelihoods | Reward hacking on inconsistent preferences; representational failures |

For the boundary failure I was facing: SFT could not fix it because the training distribution did not provide enough boundary examples to push the model past the ambiguity. DPO could fix it — *if* paired preference data is available and *if* the model has the representational capacity to express the distinction.

That second condition is where the analysis turned.

---

## The Diagnostic That Actually Matters

Before reaching for DPO, the right question is: is this a preference problem or a representation problem?

From Day 3 research on LoRA rank: a rank-16 adapter has a hard ceiling on the number of independent directions it can move the model. The boundary failure at conf≈0.50 lives in a thin slice of the decision surface — possibly a slice that lies outside the rank-16 subspace entirely. If that is the case, then no preference signal will help. DPO cannot add representational capacity to the adapter — it can only reshape the distribution within the existing subspace.

The diagnostic ordering: fix the ceiling first. Increase rank to 64 with k_proj added to target modules. Re-run the evaluation. If boundary failures persist after that change, then preference data plus DPO is the right next lever. Reaching for DPO before fixing the rank ceiling is reaching for the wrong tool — and risks wasting the preference data you collected on a model that cannot represent the distinction the preferences encode.

---

## ORPO: One Step Further

ORPO (Hong et al., 2024) integrates SFT and preference alignment into a single training pass. Instead of SFT first, then DPO on the SFT checkpoint, ORPO adds an odds-ratio penalty directly to the SFT objective:

```
L_ORPO = L_SFT - λ · log(odds_ratio(y_chosen, y_rejected))
```

The odds ratio penalizes the model for assigning similar probability to chosen and rejected outputs during the same forward pass that also maximizes likelihood of chosen. This eliminates the two-stage training pipeline and avoids the reference-model overhead.

For production use, ORPO is appealing: half the GPU memory (no reference-model copy), faster single-pass training, no separate alignment step. The trade-off: the `β` parameter in DPO is more interpretable and gives explicit control over how far the model can shift from its pre-alignment behavior. ORPO trades some of that interpretability for operational simplicity.

---

## The One-Line Summary

DPO and SFT solve different problems. SFT teaches the model what to produce; DPO teaches the model to prefer one output over another. DPO's gradient is highest where SFT's gradient is silent — at the boundary between similarly-likely outputs. But DPO cannot fix what the architecture cannot express. Diagnose the failure mode first: if it is representational (rank too low, modules frozen), fix that. If it is preferential (capacity exists, no contrast signal was given), reach for DPO.

For my own next training run: rank=64 with k_proj first. If boundary failures survive that change, then collect paired preferences on the boundary inputs and apply DPO with β tuned conservatively. ORPO is the single-pass alternative if I want one fewer training stage in the production pipeline.

---

*Sources: Rafailov et al. NeurIPS 2023 "Direct Preference Optimization: Your Language Model is Secretly a Reward Model"; Hong et al. 2024 "ORPO: Monolithic Preference Optimization Without Reference Model"; Ouyang et al. 2022 "Training language models to follow instructions with human feedback".*
