---
name: publish-check
user-invocable: true
allowed-tools: Read, Write, Glob, WebSearch, AskUserQuestion
description: Validate today's explainer and thread against the Week 12 publication checklist before they ship under your identity. Run this before the evening call or before pushing to GitHub. Blocks submission if critical checks fail.
---

# Publish Check

## Trigger
`/publish-check`

## Purpose
Runs the 6-item publication checklist against `explainer.md` and `thread.md`. Surfaces any failures with specific fixes before your work ships publicly.

---

## Workflow

### Step 1: Load Files

Use AskUserQuestion to ask:
1. Path to today's pair folder (e.g., `pair_DAY_1/`)

Read all of:
- `{folder}/explainer.md`
- `{folder}/thread.md`
- `{folder}/question.md` (to verify alignment)
- `{folder}/sources.md` (if it exists)

### Step 2: Run All 6 Checks

Run every check. Do not skip any. For each check, produce a PASS / FAIL / WARN verdict with specific evidence from the files.

---

**Check 1 — Two canonical sources cited with links**

Pass condition: At least 2 sources are cited in the explainer. Each must be:
- A link to an original paper (arxiv, ACL, NeurIPS, ICML) OR authoritative documentation (official model card, library docs, technical report)
- NOT a blog post, Medium article, or second-hand summary as a load-bearing citation (it's ok as a follow-on pointer, not as the evidence for a claim)

How to check: Search the explainer text for URLs and citation patterns. For each URL found, note whether it appears to be a primary source or a secondary one.

Flag: "Second-hand summary used as load-bearing citation" if a blog or newsletter is the only evidence for a key claim.

---

**Check 2 — Code or concrete demonstration present**

Pass condition: The explainer contains at least one of:
- A runnable code block (Python or similar)
- A concrete worked example with specific numbers
- A diagram description that could be reconstructed by the reader

How to check: Look for ``` code blocks or numbered worked examples.

Flag if: The explainer only describes the mechanism without showing it.

---

**Check 3 — Thread reads standalone**

Pass condition: Each tweet in `thread.md` makes sense to a reader who never reads the blog. Check by reading each tweet in isolation.

How to check: For each tweet, ask: "If this were the only tweet I saw, would I come away with something correct and useful?"

Flag: Any tweet that assumes the reader saw a previous tweet to make sense.

---

**Check 4 — Asker sign-off received**

Pass condition: Check for either:
- A completed `signoff.md` with "Closed" or "Partially closed" checked
- Or ask the user directly if sign-off has been given verbally

If `signoff.md` does not exist or is empty: WARN — "Sign-off file missing. Get partner's verbal confirmation and fill signoff.md before submitting."

---

**Check 5 — Attribution clean**

Pass condition: Every cited paper, tool, and source in the explainer is:
- Named explicitly
- Credited to original authors/teams (not just a URL)
- Not fabricated (no paper titles that sound plausible but can't be verified)

How to check: For each citation in the explainer, note the title and URL. Flag any that cannot be verified with a quick search. Flag any citation pattern like "[Author et al., year]" where no URL is provided.

IMPORTANT: Do not just trust that URLs look correct — note any citation you cannot verify and flag it for the human to check manually.

---

**Check 6 — No factual errors visible in spot check**

Pass condition: Pick the 3 most specific technical claims in the explainer (e.g., numbers, mechanism descriptions, comparisons between approaches). For each:
- Is the claim consistent with what the cited sources say?
- Is the claim internally consistent with the code/example in the explainer?
- Is the claim falsifiable (not vague enough to be meaningless)?

How to check: Compare claims against sources.md and the code block. Flag any claim that makes a specific number or comparison without a source.

---

### Step 3: Generate Report

Format the report as:

```
## Publish Check Report — Day [N]
Date: {date}

### Results
| Check | Status | Notes |
|-------|--------|-------|
| 1. Two canonical sources | PASS/FAIL/WARN | [specifics] |
| 2. Code or demonstration | PASS/FAIL/WARN | [specifics] |
| 3. Thread standalone | PASS/FAIL/WARN | [specifics] |
| 4. Asker sign-off | PASS/FAIL/WARN | [specifics] |
| 5. Attribution clean | PASS/FAIL/WARN | [specifics] |
| 6. No visible factual errors | PASS/FAIL/WARN | [specifics] |

### Overall: [READY TO SHIP / NEEDS FIXES / BLOCKED]

### Required fixes before publishing:
[List only FAILs — specific, actionable]

### Warnings (fix if possible, won't block):
[List WARNs]
```

### Step 4: Gate

- If any check is FAIL: "BLOCKED — fix the items above before shipping."
- If all checks PASS or WARN: "READY TO SHIP — address warnings if time allows."
- If WARN on sign-off only: "Get verbal confirmation from your partner, update signoff.md, then ship."

### Step 5: Save Report

Save the report to `{folder}/publish_check.md`.

## Output
`{folder}/publish_check.md` — checklist report with PASS/FAIL/WARN per item and overall gate verdict.

## Follow-up
- Fix any FAILs, then re-run `/publish-check`
- Once READY TO SHIP: push to GitHub, publish blog, publish thread, update `thread.md` with real blog URL
