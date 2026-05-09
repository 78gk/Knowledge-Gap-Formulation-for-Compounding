# What LoRA Rank Actually Controls — And Why Your Benchmark Won't Catch It

*Day 3 of my AI engineering knowledge gap series. Topic: training and post-training mechanics.*

---

I trained a LoRA adapter on a sales agent. Rank=16, targeting q_proj and v_proj — the standard tutorial setup. The numbers came back clean: Delta B = +0.1046 (p=0.018) on a sealed held-out set of 62 tasks. I shipped the adapter and wrote the deployment recommendation.

Then I looked at the per-task results. The adapter worked — on most inputs. It worked at confidence=0.85 (assertive), it worked at confidence=0.20 (abstain). It failed reliably on inputs at confidence=0.55 — exactly the boundary between two phrasing tiers.

The failure was not random. It was structural. And the reason came down to a number I had never questioned: the rank.

---

## What Rank Actually Controls

In a standard transformer, each weight matrix W (say, the query projection q_proj) lives in a space of dimension d×k — for a 0.5B model, that could be 1024×1024 = over a million values. Full fine-tuning would update all of them. LoRA instead says: the *useful* update ΔW is approximately low-rank. So instead of updating W directly, it learns two small matrices:

```
ΔW = B · A
where B ∈ ℝ^(d×r)  and  A ∈ ℝ^(r×k)
```

The rank `r` is the bottleneck. With rank=16, ΔW can have at most 16 non-zero singular values. Geometrically, this means the adapter can only move the model's representations along **16 independent directions** in weight space, no matter how many training steps you run or how large your dataset is.

Think of it like a spotlight with r beams. You can point all 16 beams at the same region and get a very bright, focused update in that direction. But you cannot illuminate a direction that lies outside the 16-beam subspace — no amount of training data will help, because the architecture itself rules it out.

---

## Why Benchmarks Don't Reveal This

Benchmarks test a distribution. If your training tasks and your test tasks share the same underlying structure, a rank-16 adapter is often sufficient — because most of the "alignment direction" of a fine-tuning task is genuinely low-dimensional. This is the key finding from Aghajanyan et al. (2021): fine-tuning on standard NLP tasks has an intrinsic dimensionality as low as 200 even for models with billions of parameters. A rank-16 adapter covers that easily.

What benchmarks miss are the **boundary regions** — inputs that sit at the edge of two learned categories. Those edges require the model to make fine-grained, high-precision distinctions. The distinction between "may be expanding" (inquiry tier, conf=0.55) and "might be expanding" (hypothesis tier, conf=0.45) is not a broad directional shift in weight space. It is a thin slice of the decision surface, and that slice may not lie inside the rank-16 subspace the adapter learned.

The result is a model that scores well on average — because most inputs are not on the boundary — but fails reliably on the exact inputs where precision matters most.

---

## Distribution Shift Makes It Worse

Distribution shift doesn't have to mean a completely different domain. It can just mean your test inputs hit parts of the input space that were underrepresented at training time. For a confidence-calibration task, "underrepresented at training time" often means "the training examples didn't include enough near-boundary cases" — inputs where the confidence signal sits within a narrow margin of the decision threshold.

The low-rank adapter learned a coarse approximation of the decision boundary. It works well for inputs far from the boundary because those are easy, high-signal cases. Near the boundary, the approximation breaks down. The adapter has used its 16 available directions to get the broad picture right, and it has no remaining capacity to represent the fine-grained edge.

This is the structural reason small-rank adapters can achieve strong aggregate benchmark scores while producing systematic failures on precisely the cases that matter in production.

---

## Three Adjacent Concepts Worth Knowing

**The α/r scaling ratio.** LoRA does not apply ΔW directly — it scales it by α/r before adding it to W. With α=32 and r=16, the scaling factor is 2.0. This amplifies the adapter's update, which helps early training converge faster. But it doesn't change the *subspace* — it only changes the magnitude of the update within that subspace. Rank and alpha solve different problems: rank sets the ceiling on expressiveness; alpha controls the learning rate within that ceiling.

**Target module choice.** Targeting only q_proj and v_proj means k_proj (which determines *what* patterns attention matches against) and o_proj (which mixes the attended values back into the residual stream) are frozen. For tasks requiring fine-grained numerical reasoning, k_proj may be load-bearing — it controls which contexts activate which attention heads. Leaving it frozen means the model's attention pattern is anchored to pre-training behavior even as q and v shift.

**Optimization vs representation.** This is the cleanest framing of the failure. An optimization failure is when training fell short — more data, more steps, a better learning rate would help. A representational failure is when the architecture cannot express the correction at all. Boundary failures at low rank are representational. No amount of additional training will resolve them. The fix is structural: increase rank, change target modules.

---

## The One-Line Summary

LoRA rank is a hard ceiling on the number of independent directions the adapter can learn. Benchmarks measure average performance across a distribution, so they reward adapters that get the broad directions right. Boundary failures live in the directions the rank budget didn't reach — and distribution shift just means your deployment inputs are hitting those unreached directions more often.

The practical recipe: benchmark first, then check the edges. If boundary failures show up after deployment, the first config change is rank (try 64–128 with α=r), not alpha. And add k_proj to target modules if the task requires fine-grained attention pattern matching.

---

*Sources: Hu et al. 2021 "LoRA: Low-Rank Adaptation of Large Language Models"; Aghajanyan et al. 2021 "Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning"; Biderman et al. 2024 "LoRA Learns Less and Forgets Less".*
