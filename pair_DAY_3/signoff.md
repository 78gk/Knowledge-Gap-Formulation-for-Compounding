# Gap Closure Sign-off — Day 3
**Asker:** Kirubel Tewodros
**Explainer written by:** Kirubel Tewodros (for Melaku's question) / Melaku Yilma (for Kirubel's — pending)
**Date:** 2026-05-07 (Thursday)

## Gap Closure Judgment

[x] Closed

## What I Understand Now That I Didn't Before

Before today I knew the LoRA config (rank=16, alpha=32, q_proj+v_proj) produced a boundary failure at conf≈0.50, but I had no explanation for why. I now understand that rank is a hard ceiling on the number of independent directions the adapter can move the model — not a tuning knob that training data can compensate for. The subspace is fixed by architecture. This means the adapter could learn broad phrasing-tier patterns (high confidence → assertive, very low → abstain) but had no remaining capacity to represent the thin decision surface at the exact inquiry/hypothesis threshold. Separately, targeting only q_proj and v_proj leaves k_proj frozen, which means the model's attention pattern matching stays anchored to pre-training behavior even as query and value projections shift. For a task requiring fine-grained numerical calibration, that frozen attention routing is a structural limitation — not a data or learning-rate problem. Alpha, by contrast, only scales the update magnitude within the already-constrained subspace and would not fix a boundary failure regardless of its value.

## What Remains Open

None — the mechanism is resolved and the grounding edit to `model_card.md` documents the specific remediation path (rank=64, add k_proj) for production follow-up.
