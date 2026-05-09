# Evening Call Summary — Day 5
**Format:** Self-review + AI critique pass (paired-research portion of program concluded with Day 4)
**Date:** 2026-05-09 (Saturday)

## Critique Received on the Speculative Decoding Research

The strongest part of the speculative decoding analysis was the explicit identification of the size regime where the technique applies — and the clean conclusion that it does *not* apply to the 0.5B evaluator. The non-applicability is structural, not a configuration choice. Two refinements emerged from the critique pass: (1) the explanation of why the target model can verify k tokens in one forward pass (because verification is a prefill over k tokens, which is parallel) was implicit but not fully spelled out, so I added a one-line explainer on that point in the signoff; (2) Medusa-style approaches (prediction heads on the target model itself) were initially absent as an alternative for small models — the sources.md follow-on reading now flags this as the next reading.

## Critique Received on the DPO Explainer

The DPO explainer's strongest sections were the training-objective formula breakdown and the SFT-vs-DPO gradient-signal table. Two pieces of feedback led to revisions: (1) the explainer should distinguish between DPO applied on top of an SFT checkpoint (the standard production use case) versus DPO from scratch — the reference model in the DPO objective IS the SFT checkpoint, and starting without SFT means the reference is the base model, which can lead to reward hacking on early preference data. This nuance was added to the "What DPO Optimizes" section. (2) The diagnostic split (representation vs. preference problem) was refined to "fix the ceiling before shaping the distribution" — a cleaner framing that connects directly to the Day 3 rank-geometry analysis.

## Revisions Made

- Added the explicit "verification is a prefill over k tokens" mechanism note in the speculative decoding signoff
- Added Medusa as a follow-on reading in sources.md
- Clarified in the DPO explainer that DPO is a post-SFT step (the reference model in the objective is the SFT checkpoint, not the base model)
- Tightened the "diagnostic ordering" framing: rank ceiling first, preference data second

*Self-attested by: [x] Kirubel Tewodros — both research artifacts pass the four-property rubric and the public-artifact quality bar (two canonical sources cited, concrete worked example/formula derivation, attribution clean).*
