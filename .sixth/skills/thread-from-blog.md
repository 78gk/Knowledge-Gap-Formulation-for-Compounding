---
name: thread-from-blog
user-invocable: true
allowed-tools: Read, Write, AskUserQuestion
description: Convert a finished explainer blog post into a 4-6 tweet thread for Week 12 daily publishing. Use after the evening call revisions are done and the asker has signed off.
---

# Thread From Blog

## Trigger
`/thread-from-blog`

## Purpose
Compresses your revised `explainer.md` into a 4–6 tweet thread where every tweet stands alone. Saves to `thread.md`.

---

## Workflow

### Step 1: Load the Explainer

Use AskUserQuestion to ask:
1. Path to your finalized `explainer.md` (e.g., `pair_DAY_1/explainer.md`)
2. Are you writing this as a Twitter/X thread or LinkedIn post thread? (affects tone slightly)

Read the file.

### Step 2: Extract Thread Building Blocks

From the explainer, identify:
- The core question (1 sentence — sharpest possible version)
- The load-bearing mechanism (the actual answer — 2–3 sentences max)
- The most concrete demonstration (code snippet, number, or analogy that makes the mechanism visible)
- The single most valuable adjacent concept
- The production/FDE implication (why this matters for real engineering work)
- The blog URL placeholder (to be filled in when published)

### Step 3: Write the Thread

Write 4–6 tweets using the structure below. Hard constraints:
- Each tweet must stand alone — a reader who never clicks through gets a coherent, correct, compressed explanation
- Max 280 characters per tweet (Twitter) or ~400 characters per post (LinkedIn)
- No jargon without a one-phrase definition in the same tweet
- No hedging phrases ("sort of", "kind of", "basically") — be direct
- The code snippet (if used) goes in tweet 3 or 4 as a short block — 5–8 lines max

---

**Tweet 1 — The Hook (name the question)**
Structure: "[Specific thing] is one of those concepts everyone uses but almost nobody can actually explain. Here's what's really happening: 🧵"
Or: "If you've ever wondered [exact question in plain language], here's the mechanism: 🧵"
Goal: a reader who knows nothing about the topic clicks through. A reader who does know the topic feels seen.

**Tweet 2 — The Mechanism (how it actually works)**
Structure: Name the mechanism. One concrete sentence on each moving part. No abstraction beyond what's needed to be correct.
Goal: this tweet alone would answer a junior engineer's question.

**Tweet 3 — Show It (code, number, or analogy)**
Structure: Short code block or "Here's a concrete example:" followed by the worked example. End with what to notice.
Goal: the mechanism becomes visible, not just described.

**Tweet 4 — The Adjacent Concept (highest signal neighbor)**
Structure: "The reason this matters for [adjacent concept]: [one sharp sentence connecting them]."
Goal: expands the reader's mental model by one useful node.

**Tweet 5 — The FDE Implication (production/real work angle)**
Structure: "For production systems, this means: [specific implication]. Common mistake: [what engineers get wrong as a result of not knowing this]."
Goal: engineers leave knowing what to do differently.

**Tweet 6 — The Pointer (link to blog)**
Structure: "Full explainer with code + paper links: [URL] — covers [2 specific things the blog goes deeper on that the thread couldn't fit]."
Goal: gives readers who want more a clear reason to click, not just "read the blog."

---

### Step 4: Quality Check

Before saving, verify each tweet against:
- [ ] Stands alone without other tweets
- [ ] No unclosed jargon
- [ ] No factual overstatement (matches the explainer)
- [ ] Under character limit

If any tweet fails, rewrite it.

### Step 5: Save

Save to `{pair_folder}/thread.md` using this format:

```markdown
# Tweet Thread — Day [N]
**Platform:** Twitter/X [or LinkedIn]
**Topic:** [topic]

---

**Tweet 1:**
[text]

**Tweet 2:**
[text]

**Tweet 3:**
[text — include code block if applicable]

**Tweet 4:**
[text]

**Tweet 5:**
[text]

**Tweet 6:**
[text — URL placeholder: INSERT_BLOG_URL]

---
*Replace INSERT_BLOG_URL before publishing.*
```

## Output
`{pair_folder}/thread.md` — ready-to-publish thread, pending blog URL.

## Follow-up
- `/publish-check` — final validation before submission
- Replace `INSERT_BLOG_URL` after publishing the blog post
