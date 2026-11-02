---
name: revising-requirements-docs
description: Records a product decision in a living requirements Google Doc that keeps one tab per revision — duplicating the latest tab, rewriting the affected sentences in place with a dated annotation, and logging the decision in its Decisions section, scoped to the new tab only. Use when a Slack thread or ticket changes what the requirements doc says, or for "actualizá el doc de requerimientos", "update the requirements doc", "reflejá esto en el doc", "agregá una versión al doc", "el doc dice X pero decidimos Y".
---

# Revising requirements docs

Produces a new revision tab in a requirements Google Doc that reflects a
decision taken elsewhere (a Slack thread, a ticket, a meeting), leaving every
earlier tab frozen as the record of what the spec said at the time. The doc is
the team's source of truth, so a revision is only written from a decision the
team actually took, never from the agent's own judgment about what the spec
should say.

Which document, who owns it and the ids of its tabs live in `config.local.md`
next to this file (see **Configuration**). Ask for anything missing.

## Revision convention

- **One tab per revision, newest first.** Tab name: `vN-Mon DD-Short title`,
  for example `v7-Nov 2-Depth cap enforced by the server`. Earlier tabs are
  never edited.
- **Header of every tab**: the doc title, the original `Mon DD, YYYY · @Author`
  line (kept as is), then a yellow-highlighted one-line summary starting with the
  version: `vN-Mon DD-<what changed and why in one line>`. When the decision
  came from a Slack thread, the summary links to it.
- **Changed sentences are rewritten in place**, each carrying an annotation
  with the decision date: `(Nov 2)`, or `(Nov 2; replaces the rule from Oct 23)`
  when it reverses an earlier dated rule. Removals are recorded in the sentence,
  not silently deleted: "the nesting hint was dropped Nov 2".
- **Every decision gets a bullet** at the end of the "Decisions and deliberate
  deviations" section: bold label, then what was decided, when, by whom
  (roles or teams, not names), why, and the ticket key.
- Dates are absolute month-day in the doc's language (English). The chat with
  the user may be in another language; the doc is not.

## Workflow

1. **Get the decision source.** The thread, ticket or message where the team
   agreed. Quote it back to the user in one line and confirm that is the
   decision to record. If the source shows a question still open, stop: there
   is nothing to write yet.
2. **Read the doc** with `Google Docs:read_doc`. The result is large (over 1 MB
   for a mature doc) and lands in a file; parse it with Python, never by eye.
   Take the current `revisionId`, the tab list, and dump the latest tab's
   paragraphs with their index ranges and text runs. Run the snippet in
   `dump-tab.md` for this.
3. **Find every passage the decision touches.** Grep the dump for the terms of
   the decision and for their consequences in other sections (a rule stated
   under Folders is often repeated under the toolbar, the menus, the tooltips
   and the platform sections). List them all before writing anything.
4. **Ask the user to create the tab.** The Docs API cannot create or duplicate
   tabs. Ask the user to duplicate the latest tab from its ⋮ menu, rename it
   with the convention above and drag it to the first position. Wait.
5. **Propose the exact text**, passage by passage: the sentence being replaced
   and the sentence that replaces it, plus the new Decisions bullet and the new
   yellow header line. Name any passage deliberately left alone and why. The
   doc is shared with the team, so this goes to the user before anything is
   written, and it is written only after an explicit go.
6. **Re-read the doc** to pick up the new tab id and the current `revisionId`,
   then apply everything in **one** `Google Docs:update_doc` call with
   `writeControl.requiredRevisionId` set, so a concurrent edit fails the batch
   instead of being overwritten. Every request is scoped to the new tab:
   - Sentence rewrites: `replaceAllText` with `tabsCriteria.tabIds` and
     `matchCase: true`, matching the exact current text copied from the dump.
     The replacement inherits the style of the first matched run, which is what
     makes the yellow header line stay yellow.
   - New Decisions bullet: `insertText` at the index just before the trailing
     newline of the last bullet, with the text starting with `\n`, so it joins
     the same list; then `updateTextStyle` with `bold: true` over the label.
   - Put the index-based requests first and the `replaceAllText` ones after:
     replacements change lengths, and indices computed from the dump are only
     valid before them.
7. **Verify by re-reading**: every edited passage shows the new text in the new
   tab, the Decisions bullet is bold-labelled and in the list, and no other tab
   contains the new date or wording. Report the result to the user with that
   check stated.
8. **Offer the follow-ups**, without doing them unasked: a reply in the source
   thread saying the doc is updated, and a comment or transition on the ticket.

## Guardrails

- Never edit a tab other than the new one, whatever the user's shortcut
  suggests; if they want an older tab changed, ask them to confirm that
  specifically.
- Never write a decision the source does not show. When the thread and the
  ticket disagree, the doc records neither until the user says which won.
- Never delete a rule silently: the new sentence says what was dropped and
  when.
- Never put colleagues' names in the doc; use roles or teams ("web and mobile
  agreed").
- Never apply without `requiredRevisionId`, and never apply before the user
  has seen the exact text.
- The `(Mon DD)` annotation is the date of the decision, not the date of the
  edit, when the two differ.

## Configuration

`config.local.md` beside this file, gitignored, holds:

```markdown
# <Doc name>
- document id: <Google Docs id>
- owner: <who owns the doc and should be told about a new revision>
- decisions section heading: <exact heading text>
- tabs: vN <tabId>, vN-1 <tabId>, ... (newest first; append on each revision)
- notes: <anything the convention above does not cover for this doc>
```

If the file is missing or the doc is not in it, ask the user for the link and
record the ids after the first revision.

## Before finishing

- [ ] The decision came from a source the user confirmed, not from inference
- [ ] The user duplicated the tab; nothing was written to an older tab
- [ ] The user saw the exact replacement text and said go
- [ ] One batch, with `requiredRevisionId`, scoped to the new tab
- [ ] Re-read confirms the edits and their absence from other tabs
- [ ] `config.local.md` updated with the new tab id
- [ ] Follow-ups (thread reply, ticket) offered, not assumed
