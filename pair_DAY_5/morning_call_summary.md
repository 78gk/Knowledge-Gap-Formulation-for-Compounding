# Morning Call Summary — Day 5
**Partners:** Kirubel Tewodros & Bereket Haile
**Date:** 2026-05-09 (Saturday)

## What Was Ambiguous in the Original Drafts

Kirubel's draft question asked "does speculative decoding apply to my evaluator?" but conflated three separate sub-questions: what speculative decoding is, whether it works on small models, and whether it's the right optimization for this specific bottleneck. Bereket's question asked about DPO vs SFT but was framed as "which is better?" — a comparison question without artifact grounding, making it too generic for a rigorous explainer.

## How Each Question Was Sharpened

Bereket pushed back on Kirubel's question: "What would change in your artifact if the answer is 'no'?" This forced a reframe — the real value is understanding when speculative decoding applies and why it doesn't apply to small target models, not just a binary yes/no. The question was revised to make the hardware/model-size regime the central constraint. Kirubel asked Bereket to add one sentence grounding his DPO question in his own Week 10 or 11 artifact — his final question grounds it in his own SFT-trained adapter's calibration failures, asking whether DPO would have fixed what SFT couldn't.

## Final Committed Questions

- Your question: In my Week 11 evaluator (0.5B model, greedy decode, T4, 256 decode steps), does speculative decoding address the sequential decode bottleneck — and if not, what does the mechanism tell me about what the right lever is?
- Partner's question: I trained a LoRA SFT adapter on my Week 11 agent and it improved aggregate performance but still showed calibration failures on edge inputs. DPO and ORPO are listed as post-SFT alignment steps. What does DPO actually optimize — how is its training objective different from SFT — and would it have addressed calibration failure specifically, or is it solving a different problem?

*Confirmed by both partners: [x] Yes*
