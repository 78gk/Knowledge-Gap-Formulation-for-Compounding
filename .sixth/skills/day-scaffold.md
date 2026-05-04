---
name: day-scaffold
user-invocable: true
allowed-tools: Read, Write, Glob, AskUserQuestion
description: Create the daily pair folder with all 7 required Week 12 deliverable files, pre-templated and ready to fill. Use at the start of each day or immediately after committing your question.
---

# Day Scaffold

## Trigger
`/day-scaffold [N]`

## Purpose
Creates `pair_DAY_N/` folder with all 7 required deliverable files as templates. Prevents missing files at the 23hr UTC submission deadline.

---

## Workflow

### Step 1: Get Day Info

If `N` was not passed as an argument, use AskUserQuestion to ask:
1. What day number is this? (1–6)
2. What is today's topic?
3. What is your partner's name?

### Step 2: Create Folder

Create directory: `pair_DAY_{N}/`

### Step 3: Create All 7 Files

Create each file with a template. Fill in `{day}`, `{topic}`, `{partner}`, `{date}` from what you know.

---

**`pair_DAY_{N}/question.md`**
```markdown
# Research Question — Day {day}
**Topic:** {topic}
**Date:** {date}

## Question
[Your final sharpened question — fill after morning call or use /question-sharpener]

## Artifact Connection
[Name the specific Week 10 or 11 artifact and the exact section this connects to]

## Why This Gap Matters
[One sentence on what would change in the artifact if this gap were closed]
```

---

**`pair_DAY_{N}/morning_call_summary.md`**
```markdown
# Morning Call Summary — Day {day}
**Partners:** [Your name] & {partner}
**Date:** {date}

## What Was Ambiguous in the Original Drafts
[3–5 sentences describing what was unclear or weak in each partner's draft question]

## How Each Question Was Sharpened
[Describe the specific changes made during the call]

## Final Committed Questions
- Your question: [paste final question]
- {partner}'s question: [paste their final question]

*Confirmed by both partners: [ ] Yes*
```

---

**`pair_DAY_{N}/explainer.md`**
```markdown
# Explainer — Day {day}
**Question answered:** [{partner}'s question — paste here]
**Written by:** [Your name]
**Date:** {date}

---

[600–1,000 words. Structure: open with the question anchored concretely → name the load-bearing mechanism → show it (code/diagram/worked example) → connect 2–3 adjacent concepts → pointers to papers and tools]

---

## Sources
- [Paper/source 1](url)
- [Paper/source 2](url)
- Tool used: [name + what you ran]
```

---

**`pair_DAY_{N}/thread.md`**
```markdown
# Tweet Thread — Day {day}
**Topic:** {topic}
**Date:** {date}

---

**Tweet 1 (hook — name the question):**
[Tweet text — standalone, no jargon without definition]

**Tweet 2 (mechanism — how it actually works):**
[Tweet text]

**Tweet 3 (mechanism continued or visualization/code snippet):**
[Tweet text]

**Tweet 4 (adjacent concept with highest signal):**
[Tweet text]

**Tweet 5 (production implication or why it matters for FDE work):**
[Tweet text]

**Tweet 6 (link to blog):**
[Tweet text — links to published explainer]

---
*Each tweet must stand alone if the reader doesn't click through.*
```

---

**`pair_DAY_{N}/evening_call_summary.md`**
```markdown
# Evening Call Summary — Day {day}
**Partners:** [Your name] & {partner}
**Date:** {date}

## Feedback You Gave {partner} on Their Explainer
[Specific feedback — what landed, what didn't, what assumed too much prior knowledge]

## Feedback You Received on Your Explainer
[What {partner} said about your explainer — specific, not general]

## Revisions Made After the Call
[What you changed in your explainer based on feedback]

*Written by: [ ] [Your name] [ ] {partner}*
*Confirmed by other partner: [ ] Yes*
```

---

**`pair_DAY_{N}/signoff.md`**
```markdown
# Gap Closure Sign-off — Day {day}
**Asker:** [Your name]
**Explainer written by:** {partner}
**Date:** {date}

## Gap Closure Judgment
[ ] Closed
[ ] Partially closed
[ ] Not closed

## What I Understand Now That I Didn't Before
[One paragraph — be specific about the mechanism you now understand and how it changes your mental model]

## What Remains Open
[If partially/not closed: what specific sub-question or mechanism still needs work]
```

---

**`pair_DAY_{N}/grounding_commit.md`**
```markdown
# Grounding Commit — Day {day}
**Date:** {date}

## Artifact Edited
[File name and path in your Week 10 or 11 portfolio]

## What Changed
[Paste the before and after — the exact paragraph, probe, code section, or memo sentence that was edited]

## Why It Changed
[One paragraph explaining what you now understand that caused this edit — grounded in what the explainer revealed]

## Commit Reference
[Git commit hash or PR link once committed]
```

---

**`pair_DAY_{N}/sources.md`**
```markdown
# Sources — Day {day}
**Compiled by:** [Your name]
**Date:** {date}

## Canonical Papers Read
1. [Title](url) — [One sentence on what this paper establishes relevant to the question]
2. [Title](url) — [One sentence]

## Tool or Pattern Used
- **Tool:** [Name]
- **What you ran:** [Describe the experiment or code you executed]
- **What it revealed:** [What the output showed about the mechanism]

## Follow-on Reading (optional)
- [Title or resource](url) — [Why worth reading next]
```

---

### Step 4: Confirm

Report back:
```
Created pair_DAY_{N}/ with 7 files:
  question.md
  morning_call_summary.md
  explainer.md
  thread.md
  evening_call_summary.md
  signoff.md
  grounding_commit.md
  sources.md
```

## Output
`pair_DAY_{N}/` folder with all 7 templated deliverable files.

## Follow-up
- `/gap-analyst` — surface your gap candidates for today's topic
- `/explainer-draft` — research and draft your explainer for your partner's question
- `/publish-check` — validate before submitting
