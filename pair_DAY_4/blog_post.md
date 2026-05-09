# What a Confidence Interval Tells You That a P-Value Doesn't — And Why It Changed My Deployment Gate

*Day 4 of my AI engineering knowledge gap series. Topic: evaluation and statistics.*

---

In my Week 11 ablation memo I reported a number: Delta B = +0.1046 with p=0.018. I called the result statistically significant and moved on. Then I set the production deployment gate at +0.009 — the lower bound of the 95% confidence interval [+0.009, +0.205].

I copied that pattern from a template I had read. I could not have explained, under questioning, what the CI width was telling me, why I chose the lower bound rather than the point estimate, or whether n=62 was even enough to support the deployment decision I was making.

Here is what I had to learn before I could defend my own analysis.

---

## Two Numbers, Two Different Questions

A p-value answers exactly one question: how unlikely is this result if there is actually no effect? My p=0.018 says: only 1.8% of the time would I see Delta B at least this large by chance alone. The threshold for "statistical significance" is conventionally p < 0.05 — my result clears it.

That is all the p-value tells me. It does not tell me how big the effect is. It does not tell me how reliably I have measured the effect. It does not tell me whether the difference is large enough to justify deployment.

A confidence interval answers a different question: given the data, what range of true effect sizes is plausible? My 95% CI [+0.009, +0.205] says: under standard inferential assumptions, the true effect could plausibly be anywhere from nearly zero (+0.009) to nearly twice my point estimate (+0.205). That width — 0.196 on a [0,1] scale — is the honest measurement of how much I do not know.

Two numbers. Two different questions. Conflating them is the source of the gap.

---

## Why the CI Width Is Big — And Why That's Not Bad

A wide CI on a small held-out set is expected, not a failure. With n=62 evaluation tasks and paired bootstrap, the variance in the estimate is structural. The width is bounded below by the inherent uncertainty of evaluating a stochastic process on a finite sample. To halve the CI width I would need approximately 4× the data — n≈250 tasks rather than 62.

The width is not telling me my result is weak. It is telling me how precisely I have measured the result I have. Treating wide CI as a sign of failure leads to two equally bad outcomes: either rejecting genuinely useful results because the CI is wide, or padding the evaluation set without understanding what additional precision actually buys.

The right read of [+0.009, +0.205]: I am 95% confident the true effect is positive, and I have measured it with low precision. Both statements are true and both matter for what I should do next.

---

## Why I Set the Gate at the CI Lower Bound

The deployment gate at +0.009 is intentional, not arbitrary. The logic: in the worst plausible case consistent with my data, the adapter still beats the prompt-only baseline by at least +0.009. If the adapter cannot deliver at least this lift in production, I cannot distinguish it from random noise — and the operating cost of running it is no longer justified.

This is "uncertainty-aware deployment." It is a legitimate use of the CI lower bound. But it requires understanding what the gate is doing — not just copying a number from a table. A reviewer reading my memo without this framing might conclude I think the adapter's expected lift is +0.009, which would be a misread. The expected lift is +0.1046; the gate is set at +0.009 because that's the worst case I can defend.

The framing also has consequences for ongoing monitoring. A gate at the CI lower bound is a "pilot threshold" — it is conservative for promotion but it is not a strong production-safety floor on its own. It needs to be paired with live guardrails: rolling sample size, trend monitoring, automatic rollback if reply rate drops below the threshold for N consecutive days. The number alone is not the safety mechanism.

---

## When to Pair the Bootstrap (And When Not To)

Closely related: the same evaluation pipeline reports several deltas, not all of which need paired bootstrap. The decision rule is structural, not stylistic.

Comparisons that need paired bootstrap: any case where the same evaluation units appear in both conditions. My Delta B is a paired comparison — same 62 tasks evaluated under condition A (prompt-only) and condition B (LoRA adapter). Pairing exploits the fact that per-task difficulty is a real confounder; ignoring it inflates variance for no good reason. At n=62, the difference between paired and unpaired bootstrap is often the difference between a CI that excludes zero and one that doesn't.

Comparisons that should be reported descriptively: latency and cost are system properties, not per-task outcomes. There is no meaningful within-pair correlation to exploit. Reporting "95% CI" on a latency delta overstates inferential precision. The right format is mean ± std or a percentile range, no p-value needed.

---

## What Changed in My Memo

The memo's Production Recommendation section now carries an explicit "CI width interpretation" paragraph that distinguishes the p-value (evidence against the null) from the CI width (precision of the estimate). The deployment gate at +0.009 is framed as "uncertainty-aware" — the adapter must demonstrate at least the worst plausible effect size before full promotion — not as a claim that +0.009 is the expected lift. The latency and cost deltas in the same section now report mean ± std rather than CI bounds. None of the original numbers changed; what changed is that I can now defend them.

The lesson generalizes: every number I report from now on needs to be a number I can explain. P-value and CI answer different questions. Pair when the structure demands it. Pick deployment thresholds intentionally, knowing what they communicate. The math is not the hard part — the discipline of using the right tool for the right question is.

---

*Sources: Efron & Tibshirani 1993 "An Introduction to the Bootstrap"; Dror et al. ACL 2018 "The Hitchhiker's Guide to Testing Statistical Significance in NLP"; Bouthillier et al. 2021 "Accounting for Variance in Machine Learning Benchmarks".*
