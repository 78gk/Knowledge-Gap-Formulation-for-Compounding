# Morning Call Summary — Day 3
**Partners:** Kirubel Tewodros & Melaku Yilma
**Date:** 2026-05-07 (Thursday)

## What Was Ambiguous in the Original Drafts

Kirubel's draft question bundled three separate knobs — rank, alpha, and target modules — as equally likely causes of the boundary failure, without committing to a hypothesis. This made it too broad to research cleanly. Melaku's question was conceptually solid but had no grounding in his own Week 10/11 artifact, making it a generic LoRA question rather than a diagnostic one.

## How Each Question Was Sharpened

Melaku flagged that alpha (which controls update magnitude, not subspace) is unlikely to cause a boundary-specific failure. The question was narrowed to rank and target-module selection as the two structural constraints on what the adapter can learn. The framing was also elevated from "which knob caused this?" to "what does representational geometry say about which behavioral corrections are possible?" — following Melaku's suggested sharpening via Slack. Melaku was asked to add one artifact-grounding sentence to his own question before submitting.

## Final Committed Questions

- Your question: In Week 11 I fine-tuned Qwen2.5-0.5B-Instruct with LoRA (rank=16, targeting only q_proj and v_proj). The adapter improved overall but consistently failed at the confidence≈0.50 boundary between phrasing tiers — a case the benchmark never caught. How do rank and target-module selection constrain which behavioral corrections are actually representable during fine-tuning, and why does that produce failures near ambiguous boundaries while aggregate scores still look fine?
- Partner's question: How does LoRA rank constrain what information can actually be learned during fine-tuning, and why can small-rank adapters preserve benchmark performance while still failing on distribution-shifted tasks?

*Confirmed by both partners: [x] Yes*
