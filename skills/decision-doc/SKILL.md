---
name: decision-doc
description: Write an analysis or findings document in Santi's decision-doc format — In one minute, Decisions to be made, Recommendations, How we got here, Appendix — and publish it as a Google Doc. Use for "write this up as a doc", "make a doc in Drive", "document this analysis", "write the findings doc", or whenever the result of an investigation has to be handed to other people to act on.
---

# Decision doc

A format for documents whose job is to get a decision made, not to record
everything that was done. The reader is a colleague who opens the doc to find
out what is being asked of them; the analysis is there to back the ask, not to
walk them through the investigation in the order it happened.

## Structure

Five sections, always in this order. Use `<h2>` for each one so the Docs
outline pane works as navigation, and `<h3>` inside the longer ones.

### 1. In one minute

Three or four bold-lead paragraphs, one claim each. Someone who reads only this
section should be able to react correctly. Lead each with the conclusion in
bold, then the number that supports it. No methodology here.

### 2. Decisions to be made

Each decision as a numbered bold question, with the owner named. These are the
things that stall if nobody reads the doc. If the analysis produced no decision
for anyone, the doc probably does not need to exist — say so rather than
inventing one.

### 3. Recommendations

What you would do, grouped by the decision it serves. Bold lead-in per
paragraph. Include preconditions and the things to verify first — a
recommendation whose risks are only discussed later reads as advocacy.

### 4. How we got here

The evidence, as support for an argument already stated, never as a chronology.
`<h3>` per question the analysis answered, in the order the argument needs
them. Say what was measured and from which source before showing numbers.

### 5. Appendix

Raw distributions, full tables, data sources and windows, caveats about the
data pipeline, where the queries and outputs live. Anything a reader may want
to audit but nobody needs in order to decide. Keeping this section fat is what
keeps the other four clean.

## Honesty requirements

These are not optional and they are the reason the format works.

- **A "What this cannot see" section** inside the evidence or just after it.
  State blind spots, overcounts, unresolved populations, and accuracy limits in
  plain terms. Say explicitly which numbers are ceilings rather than estimates.
- **A "Corrections" section** whenever a figure or a conclusion changed during
  the work. Name the withdrawn number, say what disproved it, and do not soften
  it. A doc that hides its own reversals cannot be trusted on the rest.
- **Unclassified is not clean.** When a population could not be assessed, label
  it unresolved, never innocent or fine.
- Prefer "we could not verify X" over omitting X.

## Writing for Google Docs specifically

The output is read in a fixed-width page, scanned through the outline pane, and
commented on paragraph by paragraph. That changes the craft:

- **Tables: three columns, four at the very most.** Wider tables get cramped and
  wrap badly. A five-column table is nearly always two tables, or a sentence.
- **Never place two unrelated tables side by side** to save space.
- **Prose over bullet lists** in the argument sections. Bullets are hard to
  comment on and flatten the reasoning. Keep lists for genuine enumerations.
- **Bold lead-ins** at the start of paragraphs carry the scan.
- **Paragraphs of two to four sentences.** Long blocks do not get read.
- **Horizontal rules between top-level sections** give the scroll some rhythm.
- Numbers in running text, units spelled out. Anonymise per-account examples in
  the body and point to the raw files for identifiers.

## Producing the file

Write the document as HTML and create it with the Google Drive connector:

```
mcp__claude_ai_Google_Drive__create_file
  title: <the doc title>
  contentMimeType: "text/html"
  textContent: <the HTML>
```

HTML converts to a native Google Doc with real headings and editable tables.
Use `<h1>` once for the title, then `<h2>`/`<h3>`, `<p>`, `<strong>`, `<em>`,
`<hr>`, and `<table border="1" cellpadding="8" cellspacing="0">`.

Title the doc like a report, not a task: a short noun phrase naming the
subject. Put the one-line description of what the doc covers as the first
paragraph, and the team plus date as an italic line under it.

**Updating an existing doc:** the connector's `update_file` only changes
metadata (title, folder) — it cannot replace the body. To revise, create a new
doc and trash the old one with `trash_file` so two versions do not circulate.
Say that you did it; trashing is reversible.

## Before writing

Ask for the destination folder if it matters; otherwise it lands in the root of
My Drive and can be moved later. If the analysis has an accompanying artifact
or repo files, list them in the appendix so the doc is not the only trace.
