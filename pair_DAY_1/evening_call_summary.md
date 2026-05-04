# Evening Call Summary — Day 1
**Partners:** Kirubel & Nurye
**Date:** 2026-05-04

## Feedback Kirubel Gave Nurye on Nurye's Explainer
The explainer answered the question but the code block showed the mechanism without showing the output — a reader couldn't verify the claim without running it themselves. The tweet thread's third tweet assumed the reader already understood what "sequential" meant in the context of token generation, which broke the standalone requirement. Suggested adding one sentence in tweet 3 to define why each token must wait for the previous one.

## Feedback Nurye Gave Kirubel on the Explainer
The "Show It" section used distilgpt2 as a proxy and noted it in a comment, but the explainer didn't explicitly say what would differ at production scale on a T4 with Qwen2.5-0.5B. A reader might question whether the 93% decode ratio holds for a larger, faster GPU. Also, the KV cache paragraph was accurate but brief — the connection between KV cache and the decode cost was not made explicit enough for the reader to understand why the cache matters for the 320ms number specifically.

## Revisions Made After the Exchange
1. Added a sentence in the KV cache section explicitly connecting cache reuse to the per-step cost: without the cache, each decode step would recompute attention over all prior tokens (O(n²)), making the 4.5ms per step a conservative estimate. With the cache, each step only computes attention for the new token against cached K/V pairs.
2. Added a note in the code block comment clarifying that T4 with Qwen2.5-0.5B runs faster per step (~1ms vs 4.5ms for distilgpt2 on CPU) but the decode-dominance ratio (>90%) holds across hardware because prefill parallelism scales with GPU compute while decode remains sequential.

*Written by: [x] Kirubel*
*Confirmed by other partner: [x] Nurye*
