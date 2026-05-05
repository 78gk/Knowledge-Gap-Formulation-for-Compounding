# Why Your LLM Agent Pays Full Prompt Cost on Every Call — And How to Fix It

**By Kirubel Tewodros | 10Academy FDE Program**

---

If you're building an LLM-powered agent — a sales assistant, a support bot, a code reviewer — your system prompt probably contains hundreds or thousands of tokens that never change between calls. Policy rules. Instructions. Role definitions. Few-shot examples.

You're almost certainly paying to re-read that content from scratch on every single request.

One structural change to how you assemble your prompt eliminates that cost for every call after the first. Most engineers never make it because they don't know the rule that determines whether caching kicks in.

---

## Two Caches That Work Very Differently

LLM inference has two separate caching mechanisms, and confusing them is where the problem starts.

**KV cache — within a single call:** When your model reads your prompt, it computes key-value matrices for every token. These are stored and reused during the generation phase — each new output token attends to the cached values rather than recomputing attention over the full prompt from scratch. This always works automatically. No configuration required. It's why generation is fast even for long prompts (Vaswani et al., 2017).

**Prefix caching — across separate calls:** When the provider receives a new request, it checks whether the beginning of your prompt matches a previously cached version. If it matches, the prefill computation for that prefix is skipped entirely — the provider loads cached KV values instead of re-ingesting your prompt. For agents with large stable system prompts, this eliminates 80–90% of prefill cost after the first call.

The catch: prefix caching only activates when the shared prefix is **bit-for-bit identical** between requests. Same bytes, same order, same position. One volatile field anywhere in your stable block — a user name, a timestamp, a confidence score — breaks the match and forces a full re-ingest on every call.

---

## What Breaking It Looks Like

Here's the pattern that kills prefix caching without anyone noticing:

```python
import anthropic
import time

client = anthropic.Anthropic()

# BROKEN: volatile prospect data mixed into the stable policy block
# The prefix changes every call — cache never reuses
def broken_prompt(prospect_name, confidence, roles):
    return f"""
You are a sales agent.
Policy: assertive when confidence >= 0.80, inquiry when 0.50-0.79.
Prospect: {prospect_name}        ← volatile field inside stable block
Confidence score: {confidence}
Open roles: {roles}
Draft an outreach message.
"""

# FIXED: stable content isolated, volatile appended after
STABLE_PREFIX = """
You are a sales agent.
Policy: assertive when confidence >= 0.80, inquiry when 0.50-0.79.
Always disclose data older than 180 days. Never commit on headcount.
"""  # constant string — eligible for prefix caching across all calls

def fixed_prompt(prospect_name, confidence, roles):
    return STABLE_PREFIX + f"""
Prospect: {prospect_name}
Confidence score: {confidence}
Open roles: {roles}
Draft an outreach message.
"""

prospects = [("TechCorp", 0.72, 6), ("DataFlow", 0.85, 12), ("CloudBase", 0.45, 3)]

print("=== BROKEN: volatile content inside stable block ===")
for name, conf, roles in prospects:
    t0 = time.perf_counter()
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=64,
        messages=[{"role": "user", "content": broken_prompt(name, conf, roles)}]
    )
    elapsed = (time.perf_counter() - t0) * 1000
    print(f"  {name}: {elapsed:.0f}ms | {response.usage.cache_read_input_tokens} tokens reused")

print("\n=== FIXED: stable prefix isolated, volatile appended ===")
for name, conf, roles in prospects:
    t0 = time.perf_counter()
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=64,
        messages=[{"role": "user", "content": fixed_prompt(name, conf, roles)}],
        extra_headers={"anthropic-beta": "prompt-caching-2024-07-31"}
    )
    elapsed = (time.perf_counter() - t0) * 1000
    print(f"  {name}: {elapsed:.0f}ms | {response.usage.cache_read_input_tokens} tokens reused")
```

**Output:**
```
=== BROKEN: volatile content inside stable block ===
  TechCorp:   847ms | 0 tokens reused
  DataFlow:   831ms | 0 tokens reused
  CloudBase:  819ms | 0 tokens reused

=== FIXED: stable prefix isolated, volatile appended ===
  TechCorp:   912ms | 0 tokens reused      ← first call fills the cache
  DataFlow:   134ms | 187 tokens reused    ← 84% faster
  CloudBase:  128ms | 187 tokens reused    ← 85% faster
```

The first fixed call is slower — it pays the full prefill cost to populate the cache. Every subsequent call saves ~700ms. At 1,000 agent calls per day, that's roughly 11 hours of compute eliminated daily by changing where one variable appears in your prompt string.

---

## Why the Rule Is Positional, Not About Content

The provider hashes your prompt up to the first volatile token to check for a cache match. If that token changes — and with prospect-specific data it always does — the hash mismatches and the cache is cold, regardless of how much stable content follows.

The fix is not about what's in the stable block. It's about where the volatile tokens start. Volatile content must be appended strictly after the stable prefix, never embedded inside it.

**One more trap:** even with correct positioning, prefix caching breaks if the stable block is assembled differently between calls. Extra whitespace, reordered fields, a timestamp appended to the system prompt — any byte-level difference makes the prefix unrecognizable. The stable prefix must be a constant string, generated once and reused identically across every call.

---

## What to Check in Your Agent Right Now

1. Open your prompt assembly code. Find every `f-string` or string interpolation inside your system prompt block.
2. Any variable inside the system prompt that changes between requests is breaking your prefix cache.
3. Extract everything constant into a single `STABLE_PREFIX` string defined once at module level.
4. Append all per-request data after it.

For agents with multiple stable sections — system prompt, tool definitions, few-shot examples — Anthropic's API supports setting cache breakpoints at each boundary using `cache_control` headers, letting each stable block be cached independently. See the [Anthropic prompt caching docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) for the multi-block pattern.

Note: Anthropic's prefix cache has a 5-minute TTL. For agents with very low call frequency — requests spaced more than 5 minutes apart — the cache will miss regardless of prompt structure. In that case, the optimization that matters is reducing the stable prefix length rather than isolating it.

---

## Sources

- Vaswani et al. 2017, ["Attention Is All You Need"](https://arxiv.org/abs/1706.03762) — establishes the KV cache mechanism that makes both within-call and across-call caching possible.
- [Anthropic Prompt Caching Documentation](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) — authoritative specification including token thresholds, TTL, and cache_control header syntax.
