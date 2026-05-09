# Morning Call Summary — Day 5
**Format:** Self-directed gap analysis with AI as critique partner (program's paired-research portion concluded with Day 4)
**Date:** 2026-05-09 (Saturday)

## What Was Ambiguous in the Original Drafts

Initial draft of the speculative decoding question conflated three separate sub-questions: what speculative decoding is, whether it works on small models, and whether it's the right optimization for this specific bottleneck. AI critique surfaced that the binary "does it apply?" framing was the weakest part — the actually interesting answer is "no, and here is what that tells you about the inference-optimization design space." The companion DPO question was initially framed as "DPO vs SFT, which is better?" which is a comparison question without artifact grounding, making it too generic.

## How Each Question Was Sharpened

For the speculative decoding question, the reframe was from "does it apply?" to "what does the mechanism tell me about the design space my 0.5B evaluator sits in?" — turning a binary lookup into a generalizable principle. For the DPO question, the sharpening pass added a specific ground in Week 11's LoRA boundary failure (TB-0036, conf≈0.50) so the question reads as diagnostic rather than encyclopedic, and added the explicit framing "would DPO have fixed what SFT couldn't?" which forces an answer that connects to the Day 3 rank-geometry analysis.

## Final Committed Questions

- Self-asked question (research): In my Week 11 evaluator (0.5B model, greedy decode, T4, 256 decode steps), does speculative decoding address the sequential decode bottleneck — and if not, what does the mechanism tell me about what the right lever is?
- Self-selected question (explainer practice): I trained a LoRA SFT adapter that showed calibration failures on edge inputs. What does DPO actually optimize, how is its training objective different from SFT, and would it have addressed calibration failure specifically, or is it solving a different problem?

*Confirmed against the four-property rubric (diagnostic, grounded, generalizable, resolvable): [x] Yes — both questions ground in concrete Week 11 artifacts and resolve in 600–1,000 word explainers.*
