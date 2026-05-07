# Morning Call Summary — Day 3
**Partners:** Kirubel Tewodros & Melaku Yilma
**Date:** 2026-05-07 (Thursday)

## What Was Ambiguous in the Original Drafts
[3–5 sentences describing what was unclear or weak in each partner's draft question]

## How Each Question Was Sharpened
[Describe the specific changes made during the call]

## Final Committed Questions
- Your question: In my Week 11 LoRA training (rank=16, alpha=32, q_proj+v_proj only), the adapter improved overall but still confuses inquiry-tier and hypothesis-tier phrasing when confidence sits near 0.50 (TB-0036). I chose rank, alpha, and target modules by convention. What do these three knobs actually control in a LoRA update — and which one would I change first to fix the boundary failure?
- Partner's question: How does LoRA rank constrain what information can actually be learned during fine-tuning, and why can small-rank adapters preserve benchmark performance while still failing on distribution-shifted tasks?

*Confirmed by both partners: [ ] Yes*
