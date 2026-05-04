---
name: gap-analyst
user-invocable: true
allowed-tools: Read, Write, Glob, AskUserQuestion, WebSearch
description: Analyze an existing artifact (probe, memo, model card, code, blog post) to surface knowledge gap candidates for Week 12 daily research. Use this every morning before the pair call to identify which gap in today's topic is worth a colleague's day of research.
---

# Gap Analyst

## Trigger
`/gap-analyst`

## Purpose
Surface 5 genuine knowledge gap candidates from your own existing work, triaged by research value. Produces `question_candidates.md` you bring to the morning call.

---

## Workflow

### Step 1: Get Today's Topic and Artifact

Use AskUserQuestion to ask (both at once):
1. What is today's voted topic and its subtopics? (paste from cohort announcement)
2. Which artifact do you want to examine? Options:
   - Paste text directly into chat
   - Provide a file path to read
   - Or both: "check my probe AND my model card section"

Read any file paths provided.

### Step 2: Run Three Interrogation Passes

Run all three passes against the artifact text. For each pass, use the exact prompt patterns below — replace `{topic}`, `{subtopics}`, and `{artifact_text}` only.

---

**Pass A — Artifact Defender**

```
Here is a paragraph (or section) from my existing work:

{artifact_text}

Today's topic is: {topic}
Subtopics: {subtopics}

List 5 things this text asserts or implies that I would NOT be able to defend if a senior ML engineer pushed back on them. For each one:
- Quote the exact phrase or sentence from the text
- Name the underlying mechanism being glossed over
- Rate research value: HIGH (closing this gap would concretely change how I build or explain things) / MEDIUM / LOW
```

---

**Pass B — Shallow Understanding Probe**

```
Today's topic is: {topic}
Subtopics: {subtopics}

List the 5 subtopics within this area that a Forward-Deployed Engineer is most likely to have shallow understanding of. For each:
- Name the subtopic
- Give a one-sentence test question whose answer would expose the gap
- Say which signal in my artifact below suggests I may have this gap (or note "artifact-independent" if it's a common gap regardless):

Artifact:
{artifact_text}
```

---

**Pass C — Surface vs. Depth Distinguisher**

```
I claim I understand: {topic}

Based on this artifact I wrote:
{artifact_text}

Ask me 5 questions whose answers would distinguish someone who has actually internalized the mechanism from someone who has only read about it. For each question, name what a surface-level answer looks like vs. what a deep answer includes.
```

---

### Step 3: Triage the Candidates

Collect all gap signals from the three passes. Remove duplicates. Triage each against these filters:

| Filter | Keep? |
|--------|-------|
| "I could look this up in 5 minutes" | No — fact lookup, not a gap |
| "I feel vague unease but can't name what I'd ask" | No — not sharp enough to produce a useful explainer |
| "Closing this would change how I explain or build something in my existing work" | YES |
| "A colleague could write 600–1,000 words that would close this" | YES |
| "This gap is common enough that others would benefit from the explainer" | YES |

Select the top 5 candidates that survive all filters.

### Step 4: Format and Save

For each candidate, format as:

```
## Gap Candidate [N]

**The gap:** [One sentence naming what you don't actually understand]
**Where it shows in your artifact:** [Quote the exact phrase that papers over it]
**Why it matters:** [What would change in your work if you closed it]
**Draft question:** [A question precise enough that a colleague could write 600-1,000 words answering it]
**Research value:** HIGH / MEDIUM / LOW
```

Save to `question_candidates.md` in the current working directory.

Tell the user: "Bring all 5 to your morning call. You'll commit to one after interrogating each other's drafts. The highest-rated one by research value is your suggested starting point, but your partner's push-back in the call is what finalizes it."

## Output
`question_candidates.md` — 5 triaged gap candidates with draft questions, ready for morning call sharpening.

## Follow-up
- After morning call: use `/question-sharpener` to score your finalized question before committing it
- After committing your question: use `/day-scaffold` to set up today's folder
