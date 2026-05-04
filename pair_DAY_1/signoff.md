# Gap Closure Sign-off — Day 1
**Asker:** Kirubel Tewodros
**Explainer written by:** Kirubel Tewodros
**Date:** 2026-05-04

## Gap Closure Judgment
[x] Closed

## What I Understand Now That I Didn't Before
Before today I could state the 320ms number but not decompose it. I now understand that inference time has two structurally different phases: prefill (parallel, fast, scales with GPU compute) and decode (sequential, dominates total time, scales linearly with max_new_tokens). The 320ms in my memo is approximately 90% decode — meaning my `max_new_tokens=256` parameter choice, not the model size or prompt length, was the primary driver of per-task latency. I also now understand that the KV cache is what makes decode tractable at all: without it, each of the 256 decode steps would recompute attention over all prior tokens. The concrete implication — cutting to `max_new_tokens=64` would reduce worst-case latency by ~3.8× with no change to task scores — is something I can now defend from first principles rather than state as a guess.

## What Remains Open
None — the specific question (which phase dominates, and what to change to make it faster) is fully answered with a mechanism, code, and a concrete recommendation tied back to the memo's numbers.
