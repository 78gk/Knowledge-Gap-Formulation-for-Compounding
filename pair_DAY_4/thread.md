# Tweet Thread — Day 4
**Platform:** Twitter/X
**Topic:** Evaluation and statistics — CI width vs p-value
**Date:** 2026-05-08 (Friday)

---

**Tweet 1 (hook):**
I reported p=0.018 and called my result statistically significant.

Then I set my production deployment gate at the CI lower bound — a number that barely cleared zero.

I didn't know what the CI width was telling me. Here's what I missed. 🧵

---

**Tweet 2 (mechanism):**
A p-value answers one question: how unlikely is this result if there's actually no effect?

p=0.018 means: unlikely enough. You can call it significant.

But it tells you nothing about the *size* of the effect or how *precisely* you've measured it.

That's what the confidence interval is for.

---

**Tweet 3 (the width is the signal):**
My result: Delta B = +0.1046 (95% CI [+0.009, +0.205])

The CI spans 0.196 on a 0–1 scale. The true effect could be anywhere from nearly zero (+0.009) to nearly double the estimate (+0.205).

That width isn't a formality. It's the honest statement of how much I don't know.

With n=62 tasks, that's expected — but it has real consequences.

---

**Tweet 4 (the production decision):**
I set the deployment gate at +0.009 — the CI lower bound.

The logic: if the true effect is at least this large, deploying is justified.

That's actually a defensible choice. You're saying: even in the worst plausible case, the adapter still beats the baseline.

But only if you understand that's what you're doing — not just copying a number from a table.

---

**Tweet 5 (the lesson):**
Two takeaways for any ML practitioner reporting evaluation results:

• p-value = evidence against the null. Significant doesn't mean large.
• CI width = precision of your estimate. Wide CI = high uncertainty, not bad results.

Set deployment gates from the CI lower bound intentionally — not because it's in the template.

---

**Tweet 6 (link):**
Full explainer — the mechanism, the decision table for bootstrap policy, and what "statistically honest" evaluation actually requires:

https://medium.com/@kirutew17654321/what-a-confidence-interval-tells-you-that-a-p-value-doesntand-why-it-changed-my-deployment-gate-6f5ba88f7aa0

#MachineLearning #Evaluation #Statistics #LLM

---
*Published: 2026-05-08 — https://x.com/kirubeltewodro2/status/2053155951176237418*
