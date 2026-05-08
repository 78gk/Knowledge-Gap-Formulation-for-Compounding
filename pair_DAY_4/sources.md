# Sources — Day 4
**Compiled by:** Kirubel Tewodros
**Date:** 2026-05-08 (Friday)

## Canonical Papers Read

1. [An Introduction to the Bootstrap](https://www.taylorfrancis.com/books/mono/10.1201/9780429246593/introduction-bootstrap-bradley-efron-robert-tibshirani) (Efron & Tibshirani, 1993) — canonical reference for bootstrap CIs; Chapter 9 covers the paired vs unpaired distinction and the variance-reduction argument behind pairing

2. [The Hitchhiker's Guide to Testing Statistical Significance in NLP](https://aclanthology.org/P18-1128/) (Dror et al., ACL 2018) — directly addresses when paired bootstrap is required in NLP evaluation pipelines; explains the dependency assumption in plain language

3. [Accounting for Variance in Machine Learning Benchmarks](https://arxiv.org/abs/2103.03098) (Bouthillier et al., 2021) — shows empirically that small held-out sets (n<50) produce high-variance benchmark rankings; motivates why CI width matters more than point estimates for deployment decisions

## Tool Used
- **Tool:** Claude Code with direct Week 11 artifact access
- **What you ran:** Read memo.md Sections 9.2 and 9.3, run_ablation.py, and model_card.md to ground the question in the exact CI [+0.009, +0.205] and the production gate derivation
- **What it revealed:** The gate at +0.009 is defensible as a conservative lower-bound deployment threshold — but only if understood as "the edge of statistical uncertainty," not as a strong minimum effect size. The CI width (0.196) on n=62 tasks is expected and honest, not a sign of a weak result.

## Follow-on Reading
- [Statistical Significance Tests for Machine Translation Evaluation](https://aclanthology.org/W04-3250/) (Koehn, 2004) — foundational paper on bootstrap resampling in model evaluation; shows why paired resampling outperforms unpaired for small n
