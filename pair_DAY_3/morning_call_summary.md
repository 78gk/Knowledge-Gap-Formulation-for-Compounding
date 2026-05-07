# Morning Call Summary — Day 3
**Partners:** Kirubel Tewodros & Melaku Yilma
**Date:** 2026-05-07 (Thursday)

## What Was Ambiguous in the Original Drafts
[3–5 sentences describing what was unclear or weak in each partner's draft question]

## How Each Question Was Sharpened
[Describe the specific changes made during the call]

## Final Committed Questions
- Your question: In Week 11 I fine-tuned Qwen2.5-0.5B-Instruct with LoRA (rank=16, targeting only q_proj and v_proj). The adapter improved overall but consistently failed at the confidence≈0.50 boundary between phrasing tiers. How do rank and target-module selection constrain which behavioral corrections are representable during fine-tuning — and why does that produce boundary failures while benchmark scores still look fine?
- Partner's question: How does LoRA rank constrain what information can actually be learned during fine-tuning, and why can small-rank adapters preserve benchmark performance while still failing on distribution-shifted tasks?

*Confirmed by both partners: [ ] Yes*
