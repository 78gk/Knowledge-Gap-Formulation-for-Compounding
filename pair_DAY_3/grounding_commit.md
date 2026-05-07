# Grounding Commit — Day 3
**Date:** 2026-05-07 (Thursday)

## Artifact Edited

`week-11/model_card.md` — Limitations section

## What Changed

**Before:**
```
- Paraphrase augmentation preserves phrasing tier but reduces syntactic diversity.
  Real prospect contexts may use vocabulary outside the training distribution.
```

**After:**
```
- Paraphrase augmentation preserves phrasing tier but reduces syntactic diversity.
  Real prospect contexts may use vocabulary outside the training distribution.
- Rank-16 adapter targeting only q_proj and v_proj leaves k_proj frozen. k_proj
  determines which attention patterns are matched; keeping it frozen anchors
  attention routing to pre-training behavior. The observed inquiry/hypothesis
  misclassification at conf≈0.50 (TB-0036) is consistent with this constraint:
  the rank-16 subspace captures broad phrasing-tier patterns but cannot represent
  the thin decision surface at the exact threshold. If the 14-day reply-rate
  monitor triggers a rollback, the recommended first config change is rank=64
  with k_proj added to target modules — not an alpha adjustment, which only
  affects update magnitude within the existing subspace.
```

## Why It Changed

Before today the model card listed "boundary under-learning" as an unexplained caveat. Researching Melaku's question on LoRA rank constraints revealed the mechanism: rank sets a hard ceiling on the number of independent directions the adapter can learn, and targeting only q_proj and v_proj leaves k_proj (attention pattern matching) frozen at pre-training behavior. The inquiry/hypothesis boundary at conf≈0.50 requires a fine-grained distinction that the rank-16, q/v-only subspace cannot represent. This edit replaces the vague caveat with a concrete mechanism and a specific remediation path — making the deployment recommendation actionable if the monitor triggers.

## Commit Reference

`c1e7627` — week-11 repo (main branch)
