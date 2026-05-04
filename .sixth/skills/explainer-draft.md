---
name: explainer-draft
user-invocable: true
allowed-tools: Read, Write, Glob, WebSearch, Task, AskUserQuestion
description: Research a partner's question and produce a structured explainer draft. Runs parallel searches for canonical papers, code experiments, and adjacent concepts. Use this after receiving your partner's committed question.md to start Act III research.
---

# Explainer Draft

## Trigger
`/explainer-draft`

## Purpose
Takes your partner's committed question, runs parallel research across papers + code + adjacent concepts, and assembles a structured explainer draft into `explainer.md`. You edit and own the final piece — this gives you a researched starting point, not a finished product.

---

## Workflow

### Step 1: Load the Question

Use AskUserQuestion to ask:
1. What is your partner's committed question? (paste it, or give the path to their `question.md`)
2. What is today's pair folder? (e.g. `pair_DAY_1/`)

Read the question.md file if a path was given.

Extract:
- `{question}` — the core question text
- `{topic}` — the day's voted topic
- `{artifact_context}` — what artifact the question connects to (from their question.md)

### Step 2: Launch Parallel Research Agents

Launch 3 background Task agents simultaneously. Do NOT wait for one before launching the others.

---

**Agent 1 — Paper Search**

```
## Task: Find canonical papers for this research question

Question: {question}
Topic: {topic}

Search for the 3 most relevant canonical sources using web search. Prioritize:
1. Original papers (arxiv, NeurIPS, ICML, ICLR, ACL) when they exist
2. Official documentation or technical reports when papers don't exist
3. Avoid second-hand summaries or blog posts as primary sources

For each source found:
- Title and authors
- URL (arxiv link preferred)
- Year published
- 2-sentence abstract summary
- The specific claim or mechanism in the paper that is most relevant to the question
- Whether this is a "must-read" (directly answers the question) or "context" (helps understand adjacent concepts)

Return structured results directly (do not write files).
```

---

**Agent 2 — Code Experiment Scaffold**

```
## Task: Write a minimal runnable code experiment for this question

Question: {question}
Topic: {topic}

Write a self-contained Python script that demonstrates or measures the mechanism the question asks about.

Requirements:
- Must be runnable (use only standard packages: transformers, torch, numpy, requests, or built-ins — no exotic dependencies)
- Must produce visible output that reveals the mechanism (print statements, tensor shapes, latency measurements, attention patterns, etc.)
- Should be 20–60 lines — focused, not a tutorial
- Add a 2-line comment at the top explaining what the script demonstrates and what to look for in the output

If the question is about something that requires a model too large to run locally (>7B), write the script using a small proxy model (e.g., "gpt2" or "distilbert-base-uncased") and note explicitly what would differ at production scale.

Return the script as a code block. Do not write files.
```

---

**Agent 3 — Adjacent Concept Mapper**

```
## Task: Identify 2-3 adjacent concepts that make this question worth answering

Question: {question}
Topic: {topic}

Identify the 2–3 neighboring concepts that:
1. A reader needs to understand to fully appreciate the answer
2. Connect the gap to the wider landscape of AI engineering

For each adjacent concept:
- Name it
- One sentence on what it is
- One sentence on how it connects to the main question
- Why including it makes the explainer richer (not just padding)

Also identify 1 concept that seems adjacent but would dilute the focus — explain why to leave it out.

Return structured results directly.
```

---

### Step 3: Wait for All Three Agents

Wait for all three Task agents to complete before proceeding.

### Step 4: Assemble the Draft

Merge the three research outputs into a structured `explainer.md` draft using this template. Fill in the content from the research — do not leave placeholders empty; if a section has no good content yet, write "[RESEARCH NEEDED: what to look for]" as a note to the human.

```markdown
# [Draft the title as: "How [mechanism] actually works: [specific angle from the question]"]

**Partner's question:** {question}
**Artifact context:** {artifact_context}

---

## The Question, Anchored

[1 paragraph. Restate the question in your own words. Then name why it matters specifically — not in the abstract, but in the context of the artifact the asker mentioned. Anchor concretely before generalizing.]

## The Load-Bearing Mechanism

[1–2 paragraphs. Describe how the thing actually works at the level an FDE needs to know. Plain language. No more abstraction than necessary, no less than required to be correct. Use the paper findings from Agent 1 here.]

## Show It

```python
[Paste the code experiment from Agent 2 here]
```

[2–3 sentences: what does this code demonstrate? What should the reader look for in the output?]

## The Adjacent Picture

[2–3 short paragraphs, one per adjacent concept from Agent 3. Keep each to a paragraph — enough to show the connection, not enough to hijack the focus.]

## Pointers

**Papers to read:**
- [Paper 1 title](url) — [one sentence why]
- [Paper 2 title](url) — [one sentence why]

**Tool used:**
- [Tool/library name] — [what you ran and what it showed]

**Where to go deeper:**
- [One follow-on direction for readers who want more]

---
*Draft generated from parallel research. Review all claims against the cited sources before publishing. Replace [RESEARCH NEEDED] markers with your own findings.*
```

### Step 5: Save

Save the assembled draft to `{pair_folder}/explainer.md`.

Tell the user:
- How many sources were found and which are "must-read" vs "context"
- Whether the code experiment ran against a proxy model (note what to verify at real scale)
- Which adjacent concept was flagged as "leave out" and why
- Word count of the draft
- "This is a researched starting point. Your job: verify claims against the cited papers, run the code, rewrite in your voice, and cut anything that doesn't directly serve the question."

## Output
`{pair_folder}/explainer.md` — structured research draft, 400–700 words pre-edit.

## Follow-up
- Run the code experiment manually and note what you observe
- Read at least the 2 "must-read" papers before revising
- `/thread-from-blog` — after revising, compress to tweet thread
- `/publish-check` — validate before evening call
