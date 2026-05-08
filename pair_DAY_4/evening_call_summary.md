# Evening Call Summary — Day 4
**Partners:** Kirubel Tewodros & Hiwot Beyene
**Date:** 2026-05-08 (Friday)
**Format:** Async Slack exchange (no live call)

## Feedback You Gave Partner on Their Explainer

Hiwot's explainer on bootstrap policy was well-structured and directly answered the three-point question she committed to. The decision table (paired required for Delta A/B, descriptive only for latency/cost) was the most useful part — it makes the policy immediately actionable. One thing that could have been sharper: the explanation of *why* unpairing inflates variance when the same tasks appear in both conditions. The worked example with the "hard tasks" was effective but could have used a concrete number to make the intuition stick.

## Feedback You Received on Your Explainer

Hiwot's explainer was incisive and made three precise points: (1) p-value and CI answer different questions — p-value is evidence against zero, CI is effect-size uncertainty; (2) your +0.009 gate is "edge-of-uncertainty," not conservative — it's a defensible pilot threshold but not a strong production-safety floor; (3) the CI width (~0.196) is not a weakness, it's honest uncertainty at n=62, and it means your gate needs live guardrails (minimum sample size, trend monitoring, negative-drift rollback) to avoid false decisions. Most useful: her reframing that the gate is "near uncertainty edge" — that changes how the deployment recommendation should be worded.

## Revisions Made After the Call

The grounding edit to week-11/memo.md (§9.3, commit `278b922`) already includes the distinction between p-value and CI. Hiwot's reframing suggests adding an operational note: the 14-day monitor gate at +0.009 should be explicitly labeled "pilot mode with guardrails," not "production deployment." This is a refinement to the language, not a change to the logic.

*Written by: [x] Kirubel Tewodros*
*Confirmed by other partner: [x] Yes*
