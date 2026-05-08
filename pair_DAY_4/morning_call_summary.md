# Morning Call Summary — Day 4
**Partners:** Kirubel Tewodros & Hiwot Beyene
**Date:** 2026-05-08 (Friday)

## What Was Ambiguous in the Original Drafts

Hiwot's first draft bundled four separate evaluation questions (pass^k, bootstrap policy, judge-bias audit, contamination limits) at equal weight — too broad for one rigorous explainer. Kirubel's first draft asked what a CI tells you vs a p-value, and whether the deployment gate was defensible — solid but could sharpen the "what would change" stakes.

## How Each Question Was Sharpened

Hiwot dropped pass^k, judge-bias, and contamination to follow-on scope and committed to one load-bearing question: bootstrap policy across her Week 11 pipeline — specifically which comparisons require paired bootstrap, which can be unpaired, and what minimal patch keeps her CI/p-value claims honest. Kirubel's question stayed as-is since the CI-width framing was already clean.

## Final Committed Questions

- Your question: In my Week 11 memo I reported Delta B = +0.1046 (p=0.018) and called it significant. Then I set the production gate at +0.009 — the CI lower bound — without knowing what CI width means. What does a confidence interval tell you that a p-value does not, and was +0.009 a defensible floor to deploy on?
- Partner's question: In my Week 11 tenacious-bench pipeline (run_sealed_ablation.py + model_card.md), which reported comparisons require paired bootstrap, which can use non-paired alternatives, and what is the minimal patch that keeps my CI/p-value claims statistically honest?

*Confirmed by both partners: [x] Yes*
