# Research Question — Day 4
**Topic:** Evaluation and statistics
**Date:** 2026-05-08 (Friday)
**Partner:** Hiwot Beyene

## Question

In my Week 11 memo I reported Delta B = +0.1046 (p=0.018) and called it statistically significant. Then I set the production deployment gate at +0.009 — the lower bound of the 95% confidence interval. I used paired bootstrap because the template said to. I did not know what the CI width means or what it says about how reliable +0.1046 actually is. What does a confidence interval tell you that a p-value does not — and was +0.009 a defensible floor to deploy on, or was I building a gate on the edge of uncertainty?

## Artifact Connection

Week 11 memo (memo.md), Section 9.2 "Ablation results on sealed held-out (n=62 tasks)" and Section 9.3 "Production Recommendation." The deployment gate (≥ 0.2347 = baseline 0.2258 + CI lower bound 0.009) is derived directly from a CI lower bound I reported without understanding what CI width communicates.

## Why This Gap Matters

Closing this would let me say whether the production gate I set is genuinely conservative or dangerously loose — and tell me whether n=62 tasks was enough to support a deployment decision at all.
