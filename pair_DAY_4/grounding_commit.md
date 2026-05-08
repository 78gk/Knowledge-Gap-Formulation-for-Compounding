# Grounding Commit — Day 4
**Date:** 2026-05-08 (Friday)

## Artifact Edited

`week-11/memo.md` — Section 9.3 "Production Recommendation", Evidence cited paragraph

## What Changed

**Before:**
```
**Evidence cited.** Delta B +0.1046 (p=0.018) shows the adapter significantly
outperforms un-trained Qwen2.5-0.5B on the phrasing-gate task. Inference cost
delta is $0.00/task (§9.2 Cost-Pareto). The CI lower bound (+0.009) is the
minimum defensible lift — below this floor, the adapter's confidence-calibration
benefit cannot be distinguished from noise, and the ~$2.40M/yr Signal
Over-Claiming pipeline cost [C-004] outweighs any retention upside.
```

**After:**
```
**Evidence cited.** [same as before]

**CI width interpretation.** The 95% CI [+0.009, +0.205] spans 0.196 on a
[0,1] scale, which is expected for n=62 tasks with paired bootstrap. The width
does not indicate a weak result — it is an honest statement of measurement
uncertainty at this sample size. The p-value (0.018) confirms the effect is
unlikely under the null; the CI width quantifies how precisely we have measured
it. Setting the deployment gate at the lower bound (+0.009) is intentionally
conservative: it means the adapter must demonstrate at least the worst plausible
effect size before full promotion. A reviewer should interpret this gate as
uncertainty-aware, not as a claim that +0.009 is the expected lift.
```

## Why It Changed

Before today I understood the CI lower bound as just a number to gate deployment on — I copied the pattern without knowing what it communicated. Researching Hiwot's bootstrap question revealed the distinction: a p-value answers whether the effect exists; CI width answers how precisely you've measured it. A wide CI ([+0.009, +0.205] spanning 0.196) on n=62 is not a sign of a weak result — it is the honest uncertainty at that sample size. The original memo left a reader to infer this. The edit makes it explicit so the deployment gate reads as intentionally conservative, not as a claim that the adapter's true lift is close to zero.

## Commit Reference

`278b922` — week-11 repo (main branch)
