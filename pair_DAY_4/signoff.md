# Gap Closure Sign-off — Day 4
**Asker:** Kirubel Tewodros
**Explainer written by:** Hiwot Beyene (for Kirubel's question) / Kirubel Tewodros (for Hiwot's question)
**Date:** 2026-05-08 (Friday)

## Gap Closure Judgment

[x] Closed

## What I Understand Now That I Didn't Before

Before today I reported Delta B = +0.1046 (p=0.018, CI [+0.009, +0.205]) and set the production gate at +0.009 without being able to explain what either number communicated. I now understand that p=0.018 and the CI are answering two different questions. The p-value answers: is there evidence of any effect? The CI answers: how precisely have I measured the size of that effect? A CI spanning 0.196 on n=62 tasks is not a sign of a weak result — it is the expected uncertainty at that sample size. Setting the deployment gate at the lower bound (+0.009) is defensible precisely because it means: even in the worst plausible case consistent with the data, the adapter still beats baseline. That is uncertainty-aware deployment, not overconfidence. The key thing I was missing: these two numbers serve different purposes and should be reported with different language. Conflating them — or using either without understanding the other — produces claims that look precise but aren't.

## What Remains Open

None. The mechanism is fully resolved. Hiwot's explainer clarified the exact distinction: p-value = evidence against zero; CI = effect-size uncertainty. Her reframing of the +0.009 gate as "pilot threshold, not strong production floor" is the precise language. The grounding edit to `week-11/memo.md` §9.3 (commit `278b922`) documents the p-value/CI distinction; the deployment recommendation should carry a forward note that the gate operates in "pilot mode with live guardrails," not as a standalone production-safety floor.
