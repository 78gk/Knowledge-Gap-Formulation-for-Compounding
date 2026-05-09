# Evening Call Summary — Day 5
**Partners:** Kirubel Tewodros & Bereket Haile
**Date:** 2026-05-09 (Saturday)
**Format:** Async Slack exchange

## Feedback You Gave Bereket on His Explainer (Speculative Decoding)

Bereket's speculative decoding explainer was mechanically sound and answered the question correctly: the 0.5B evaluator is already the "small model," making draft-model speculation impractical for this setup. The strongest part was the worked example showing the speedup formula — it made the acceptance rate math concrete. Two gaps: (1) the explanation of why the target model can verify k tokens in one forward pass (because the verify step is a prefill over k tokens, which is parallel) was mentioned but not fully explained — a reader who doesn't know prefill is parallel would miss why verification is fast; (2) Medusa-style approaches (prediction heads on the target model itself) were not mentioned as an alternative for small models, which would have completed the picture.

## Feedback You Received on Your Explainer (DPO)

Bereket's feedback on the DPO explainer: the DPO training objective formula was well-explained and the table comparing SFT vs DPO gradient signal was the clearest part. His main addition: the explainer should have distinguished between DPO applied on top of an SFT checkpoint (the standard use case) vs DPO from scratch (less stable, not recommended). That distinction matters in practice — the reference model in the DPO objective IS the SFT checkpoint, and starting without SFT means the reference is the base model, which can lead to reward hacking on early preference data. His reframe of the diagnostic split (representation problem vs. preference problem) as "fix the ceiling before shaping the distribution" was the most useful framing addition.

## Revisions Made After the Call

Added clarification in the explainer's "What DPO Optimizes" section that the reference model is the SFT checkpoint — DPO is a post-SFT step, not a replacement for it. The diagnostic ordering (rank first, then DPO) was already in the explainer; Bereket's feedback confirmed this was the right recommendation.

*Written by: [x] Kirubel Tewodros*
*Confirmed by other partner: [x] Yes*
