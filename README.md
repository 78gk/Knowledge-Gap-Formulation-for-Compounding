# Week 12 — Knowledge Gap Formulation
**Author:** Kirubel Tewodros
**Program:** 10Academy Forward-Deployed Engineer, Week 12
**Submission deadline:** Saturday 2026-05-09, 21hr UTC

---

## What This Repo Contains

Five days of paired daily research: each day one knowledge gap named, one explainer written, both published publicly. Days 1–4 were paired sessions with assigned cohort partners; Day 5 was a self-directed extension day to round out the topic spine after the program's paired-research portion concluded with Day 4.

```
week-12/
├── pair_DAY_1/          # Mon 2026-05-04 — Inference mechanics (KV cache & prefix caching)
├── pair_DAY_2/          # Wed 2026-05-06 — Agent / tool-use internals (structured output)
├── pair_DAY_3/          # Thu 2026-05-07 — Training & post-training (LoRA rank geometry)
├── pair_DAY_4/          # Fri 2026-05-08 — Evaluation & statistics (bootstrap CIs)
├── pair_DAY_5/          # Sat 2026-05-09 — Production patterns (speculative decoding + DPO)
├── synthesis.md         # 1,500-word week synthesis — 10 gaps closed
├── canonical_list.md    # Annotated cohort canon contribution
└── portfolio_update.md  # Week 10–12 arc for FDE hiring managers
```

Each `pair_DAY_N/` folder contains:

| File | Contents |
|------|----------|
| `question.md` | Final sharpened question + named connection to a Week 10 or 11 artifact |
| `morning_call_summary.md` | How questions were sharpened in the morning call |
| `explainer.md` | 600–1,000 word blog post written for partner's question |
| `blog_post.md` | Public-facing version of the explainer (Substack/personal site) |
| `thread.md` | 4–6 tweet thread |
| `evening_call_summary.md` | Feedback given and revisions made |
| `signoff.md` | Asker's gap-closure judgment |
| `grounding_commit.md` | Pointer to the actual edit made to a Week 10 or 11 artifact |
| `sources.md` | Two canonical papers + tool used |

---

## Daily Pairs and Topics

| Day | Date | Topic | Partner | Grounding Commit (Week 11) |
|-----|------|-------|---------|------|
| 1 | Mon 2026-05-04 | Inference mechanics — prefill/decode split, prefix caching | Nurye Nigus Mekonen | `4ac63c2` (memo §9.2 latency cost model) |
| 2 | Wed 2026-05-06 | Agent / tool-use internals — instruction-following vs constrained decoding | Natnael Alemseged | `dd2fb7e` (judge calls use `response_format`) |
| 3 | Thu 2026-05-07 | Training & post-training — LoRA rank as representational ceiling | Melaku Yilma | `c1e7627` (model_card boundary-failure mechanism) |
| 4 | Fri 2026-05-08 | Evaluation & statistics — CI width vs p-value, paired bootstrap | Hiwot Beyene | `278b922` (memo §9.3 CI width interpretation) |
| 5 | Sat 2026-05-09 | Production patterns — speculative decoding + DPO/ORPO | Self-directed (program paired portion concluded Day 4) | `5589bd2` (memo §9.2 inference-optimization design space) |

---

## Public Artifacts

### Blog Posts
| Day | Topic | URL |
|-----|-------|-----|
| 1 | Inference-time mechanics — KV cache & prefix caching | https://kirubel860202.substack.com/p/why-your-llm-agent-pays-full-prompt |
| 2 | Structured output — instruction-following vs constrained decoding | [URL — pending publication] |
| 3 | LoRA rank as representational ceiling | [URL — pending publication] |
| 4 | Bootstrap CIs — what width tells you that a p-value does not | [URL — pending publication] |
| 5 | DPO vs SFT — what each optimizes and when DPO doesn't help | [URL — pending publication] |

### Tweet Threads
| Day | Topic | URL |
|-----|-------|-----|
| 1 | Inference-time mechanics — KV cache & prefix caching | https://x.com/kirubeltewodro2/status/2051613501907403084?s=20 |
| 2 | Structured output — instruction-following vs constrained decoding | [URL — pending publication] |
| 3 | LoRA rank as representational ceiling | [URL — pending publication] |
| 4 | Bootstrap CIs — what width tells you that a p-value does not | [URL — pending publication] |
| 5 | DPO vs SFT — what each optimizes and when DPO doesn't help | [URL — pending publication] |

---

## Final Submission Documents

- [synthesis.md](synthesis.md) — Week-level synthesis: ten gaps closed, the most surprising thing learned, canonical reading list
- [canonical_list.md](canonical_list.md) — Annotated cohort-canon contribution: 10 papers, 3 tools, 3 engineering patterns
- [portfolio_update.md](portfolio_update.md) — Week 10 → 11 → 12 trajectory for an FDE hiring manager

---

## Skills Used (`.sixth/skills/`)

| Skill | When |
|-------|------|
| `/gap-analyst` | Every morning — surfaces gap candidates from own artifacts |
| `/question-sharpener` | After morning call — scores + locks in question.md |
| `/day-scaffold` | Start of each day — creates folder with all 7 files |
| `/explainer-draft` | Midday — parallel research agents produce draft |
| `/thread-from-blog` | After evening call — compresses blog to thread |
| `/publish-check` | Before pushing — 6-item publication gate |
