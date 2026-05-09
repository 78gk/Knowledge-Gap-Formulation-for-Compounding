# Portfolio Update — Week 12
**Author:** Kirubel Tewodros
**Audience:** FDE hiring manager reviewing Weeks 10–12

---

## What This Week Added to the Portfolio

Weeks 10 and 11 produced working AI systems. Week 12 produced the understanding behind them. This document describes how five concrete edits — one per day, each grounded in something I now actually understand — improve the defensibility and depth of the Week 10 and 11 work.

---

## The Five Grounding Commits

**Commit 1 — Day 1 (2026-05-04)**
- Artifact: `week-11/memo.md` Section 9.2 Cost-Pareto, latency paragraph
- What changed: Replaced "negligible latency overhead" with a full cost model: ~90% decode, ~10% prefill, max_new_tokens=256 as primary driver, actual output length ~15 tokens. Added the specific optimization: reduce max_new_tokens to 64 for ~3× latency improvement with no score change. Corrected the imprecise "weights fused at load time" language for the LoRA adapter.
- Why it's better: The original paragraph stated a number I measured without being able to explain. The revised paragraph states a cost model I can defend — and makes a specific, actionable recommendation. Any engineer reading it can understand the parameter choice and reproduce the optimization.
- See: [pair_DAY_1/grounding_commit.md](pair_DAY_1/grounding_commit.md)

**Commit 2 — Day 2 (2026-05-06) — `dd2fb7e`**
- Artifact: `week-11/generation_scripts/synthesis_generator.py`
- What changed: Added `response_format={"type": "json_object"}` to the judge model API call. The original code relied on a prompt instruction for structured output; the patch activates grammar-constrained decoding, making format failures architecturally impossible rather than just improbable.
- Why it's better: The pipeline previously had a latent failure mode — edge-case inputs caused the judge to prepend freeform text, breaking json.loads() silently. The patch closes that failure mode structurally. The 8% latency overhead is documented and the tradeoff is explicit.
- See: [pair_DAY_2/grounding_commit.md](pair_DAY_2/grounding_commit.md)

**Commit 3 — Day 3 (2026-05-07) — `c1e7627`**
- Artifact: `week-11/model_card.md` Limitations section
- What changed: Replaced the vague "boundary under-learning" caveat with a precise mechanism: rank-16 adapter targeting only q_proj and v_proj leaves k_proj frozen, creating a hard representational ceiling. The inquiry/hypothesis misclassification at conf≈0.50 (TB-0036) is consistent with this constraint. Included the specific remediation path: rank=64 with k_proj added to target modules — not an alpha adjustment.
- Why it's better: The original caveat acknowledged a failure without explaining it, making the deployment recommendation feel incomplete. The revised limitation names the architectural cause and gives a specific next step — making the recommendation actionable if the 14-day reply-rate monitor triggers a rollback.
- See: [pair_DAY_3/grounding_commit.md](pair_DAY_3/grounding_commit.md)

**Commit 4 — Day 4 (2026-05-08) — `278b922`**
- Artifact: `week-11/memo.md` Section 9.3 Production Recommendation, Evidence cited paragraph
- What changed: Added a "CI width interpretation" note explaining that the 95% CI [+0.009, +0.205] is honest measurement uncertainty at n=62, not a sign of a weak result. Made explicit that the deployment gate at +0.009 is intentionally conservative — "worst-plausible-case" — not a claim that +0.009 is the expected lift. Added the framing that this gate requires live guardrails (trend monitoring, negative-drift rollback) to operate safely.
- Why it's better: The original memo reported numbers I couldn't explain under questioning. The revised paragraph is one I can defend: I know what each number communicates, what the gate is doing, and what assumptions it requires to be valid.
- See: [pair_DAY_4/grounding_commit.md](pair_DAY_4/grounding_commit.md)

**Commit 5 — Day 5 (2026-05-09)**
- Artifact: `week-11/memo.md` Section 9.2 Cost-Pareto, inference optimization note
- What changed: Added a note explaining why speculative decoding does not apply to the 0.5B evaluator, and what that decision space looks like — identifying the size regime where speculative decoding would be the right first optimization to evaluate.
- Why it's better: The Day 1 optimization recommendation (reduce max_new_tokens) was correct but stood without context. The addition situates it within the broader inference optimization design space — a reader can now understand not just what was chosen but why the alternatives don't apply in this specific setup.
- See: [pair_DAY_5/grounding_commit.md](pair_DAY_5/grounding_commit.md)

---

## The Trajectory Across Weeks 10, 11, and 12

**Week 10** produced TheConversionEngine — a multi-agent sales system that took a prospect's hiring signals and generated calibrated outreach messages. The system made consequential design decisions: prompt assembly for a large stable policy block, tool routing through Python rather than native tool-calling, and a phrasing-gate that mapped confidence scores to message tiers. At the time of submission, those decisions were made by instinct and convention. I could describe what the system did; I could not explain the mechanisms that determined whether it would work reliably at scale.

**Week 11** added measurement and adaptation. I built a benchmark (tenacious-bench, 62 held-out tasks), fine-tuned a LoRA adapter, ran a paired bootstrap ablation, and wrote a memo with deployment recommendations. The work was technically correct — the numbers were right, the statistical test was appropriate — but I was operating at the boundary of my understanding. I reported latency, CI bounds, and adapter limitations in language I had borrowed from the template rather than language I had earned.

**Week 12** closed the gap between the work and the understanding behind it. Five concrete edits — one to TheConversionEngine's prompt architecture (prefix caching), one to the judge pipeline (constrained decoding), one to the model card (rank constraints), one to the deployment recommendation (CI semantics), and one to the cost analysis (inference optimization design space) — made the existing work defensible. The trajectory is: build, measure, understand. Not as a sequential process but as a loop — the understanding feeds back into the build.

---

## What I Would Do Differently Now

**Prompt assembly.** TheConversionEngine assembles prompts with volatile prospect signals inside the stable policy block. This breaks prefix caching on every call. The fix is a constant `STABLE_PREFIX` string defined once at module level, with per-request data appended after it. I would design this structure from the start rather than discovering it as an optimization — the cost of getting it right at design time is zero; the cost of retrofitting it at scale is a refactor.

**Judge pipeline reliability.** Every downstream parser in every pipeline I build from now on will use format-constrained API parameters rather than prompt instructions. Instruction-following is for human-facing outputs. For machine-parsed outputs — JSON fields, structured scores, tool arguments — the mechanism needs to be structural or the failure mode is invisible. One parameter, not a longer prompt.

**Evaluation design.** If I had understood CI width and paired bootstrap before Week 11, I would have started the held-out set design with a target confidence interval width rather than using the benchmark size the template implied. I would also have separated inferential comparisons (Delta A, Delta B — where paired bootstrap applies) from descriptive system properties (latency, cost — where CI language overstates precision) from the beginning. The Week 11 analysis was correct; it would have been cleaner if the design had been intentional.

**Model architecture for fine-tuning.** Rank=16 targeting q+v is a reasonable default, not a principled choice for a calibration task. Before the next fine-tuning run I would: (1) characterize the failure mode first (boundary vs broad), (2) compute whether the task's intrinsic dimensionality requires more than 16 directions, (3) include k_proj in target modules when the task requires fine-grained attention pattern changes. The LoRA config should be derived from the task geometry, not copied from the tutorial.
