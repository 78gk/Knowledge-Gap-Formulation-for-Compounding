---
name: question-sharpener
user-invocable: true
allowed-tools: Read, Write, AskUserQuestion
description: Score a draft research question against the Week 12 four-property rubric (Diagnostic, Grounded, Generalizable, Resolvable) and return a rewrite. Use this after the morning call to lock in your final question before committing it.
---

# Question Sharpener

## Trigger
`/question-sharpener`

## Purpose
Takes your draft question (or the one you're about to commit to `question.md`) and scores it 1–5 on each of the four rubric properties. Returns the score, the reason for each score, and a rewritten version that addresses weaknesses.

---

## Workflow

### Step 1: Get the Question

Use AskUserQuestion to ask:
1. Paste your current draft question.
2. Which Week 10 or 11 artifact does this question connect to? (file name or description)
3. Optional: paste 1-2 sentences of context about what you were confused about.

### Step 2: Score Against the Four Properties

Score the question on each property using the rubric below. Be direct — a weak score is useful information, not an insult.

---

**Property 1 — Diagnostic (1–5)**
Does the question name a *specific* gap whose closure changes how the asker does FDE work?

| Score | Meaning |
|-------|---------|
| 5 | Names a precise mechanism. Closing it changes something concrete in the asker's work. |
| 3 | Names a topic area but not the specific gap. Could be sharpened further. |
| 1 | Vague, theoretical, or answerable by a Wikipedia paragraph. |

---

**Property 2 — Grounded (1–5)**
Does the question connect to a specific artifact the asker has shipped?

| Score | Meaning |
|-------|---------|
| 5 | Names exact artifact + specific section/line/decision that depends on closing this. |
| 3 | Mentions a project or week but not which artifact or what specifically would change. |
| 1 | No connection to any existing work. Pure curiosity question. |

---

**Property 3 — Generalizable (1–5)**
Would closing this gap help many FDE practitioners, not just the asker?

| Score | Meaning |
|-------|---------|
| 5 | Closing the gap helps any engineer working with similar systems. Common confusion point. |
| 3 | Helpful to others in similar contexts but fairly specific to the asker's stack. |
| 1 | Idiosyncratic to the asker's exact situation or trivial to anyone else. |

---

**Property 4 — Resolvable (1–5)**
Can a thoughtful colleague write a 600–1,000 word explainer that closes it?

| Score | Meaning |
|-------|---------|
| 5 | Precise enough. One focused explainer closes it. |
| 3 | Either too broad (needs narrowing to one sub-mechanism) or too narrow (answerable in one paragraph). |
| 1 | Requires a textbook chapter OR is answered by a single sentence. |

---

### Step 3: Calculate Total and Gate

Total score = sum of all four property scores (max 20).

| Total | Status |
|-------|--------|
| 17–20 | Ready to commit |
| 13–16 | Sharpen before committing — at least one property needs work |
| Below 13 | Do not commit — rewrite required |

### Step 4: Produce the Rewrite

Write an improved version of the question that addresses any property scoring below 4. Show the change clearly:

```
ORIGINAL:
[original question]

REWRITE:
[improved question]

WHAT CHANGED:
- [Property name]: [what was changed and why]
```

If the score is already 17+, confirm it's ready and suggest the exact wording for `question.md` including the artifact pointer line.

### Step 5: Save

Use AskUserQuestion to ask: "Save this to question.md in today's pair folder? If yes, what is the folder path (e.g. pair_DAY_1/)?"

If confirmed, save to `{folder}/question.md` using this format:

```markdown
# Research Question — Day [N]

## Question
[Final question text]

## Artifact Connection
[Name the specific artifact and the exact section/decision this question connects to]

## Why This Gap Matters
[One sentence on what would change in the artifact if this gap were closed]

## Rubric Scores
- Diagnostic: [score]/5
- Grounded: [score]/5
- Generalizable: [score]/5
- Resolvable: [score]/5
- **Total: [score]/20**
```

## Output
- Score report with reasoning for each property
- Rewritten question (if needed)
- Optionally saves to `question.md`

## Follow-up
- Use `/day-scaffold` to create the full day folder if not already done
- Share your committed `question.md` with your partner
