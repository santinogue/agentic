---
name: writing-skills
description: Creates a new skill or updates an existing one, following Anthropic's skill-authoring best practices and this repo's conventions. Use when the user says "create a skill", "make this a skill", "turn this into a skill", "armemos una skill", "update the X skill", "this skill isn't firing", or when a process just worked well enough in conversation that it should be captured for next time.
---

# Writing skills

A skill is a procedure an agent follows, written for the agent. It earns its
place only when it carries something the model would otherwise get wrong:
judgment, local conventions, an order of operations, a rule about what not to do.

## Before writing anything

**Start from a real transcript, not an idea.** The best skills come from work
that just happened. Ask what the agent got wrong or needed told, and write down
those specific failures — they are the spec.

**Name the gap in one sentence.** "Without this skill, the agent does X instead
of Y." If that sentence can't be written, the skill isn't needed yet.

**Write three scenarios that would exercise it** before writing instructions, and
keep them in mind as the acceptance test. Then write the minimum that makes those
scenarios go right — not a complete treatment of the topic.

**Prefer updating an existing skill** over adding a near-duplicate. Check what is
already in `skills/` first.

## Frontmatter

Two required fields, with hard limits:

- `name`: max 64 characters, lowercase letters, numbers and hyphens only. No XML
  tags. Cannot contain the words "anthropic" or "claude". Prefer gerund form —
  `writing-skills`, `reviewing-prs`, `analyzing-spreadsheets`. Avoid `helper`,
  `utils`, `tools`, `data`.
- `description`: max 1024 characters, non-empty, no XML tags.

**The description is the trigger.** It is the only part loaded at startup, and it
is what the agent uses to pick this skill out of a hundred. Write it in **third
person** — "Creates a new skill", never "I can help you" or "You can use this".
Say both what it does and when to use it, and list the actual phrasings that
should fire it, including the ones in the language the user types in.

Good: `Extracts text and tables from PDFs, fills forms, merges documents. Use
when working with PDF files or when the user mentions PDFs, forms, or document
extraction.`

Bad: `Helps with documents.`

## Body

**Assume the model is already smart.** Only add what it doesn't have. Challenge
every paragraph: does this justify its tokens, or is it explaining something any
model already knows?

**Match freedom to fragility:**

- *High freedom* — prose steps, when several approaches are valid and context
  decides.
- *Medium freedom* — a template or a parameterized command, when a preferred
  pattern exists.
- *Low freedom* — the exact command and "do not modify it", when the operation is
  fragile, destructive, or must run in sequence.

**Encode judgment, not just steps.** The reusable part is usually a rule: who
decides, when to stop and ask, what never to do, what to do when the user pushes
back.

**Give workflows real steps**, and for long ones a checklist the agent can copy
and tick off. Where output quality depends on it, add a feedback loop: produce →
validate against a stated standard → fix → only then proceed.

**Show examples, not adjectives.** Concrete input/output pairs teach a format
better than describing it.

**Offer one default, not a menu.** Name the escape hatch in a clause, not a list
of equal options.

## Structure

Skills here share a spine, not a template. The middle takes the shape of the
task — steps for a workflow, a format and examples for a document, rules for a
review — and only these four pieces are fixed:

```
frontmatter      name in gerund form, description in third person
# Title
opening          one paragraph: what it produces and when it applies
## ...           the middle, shaped by the task
## Guardrails    what is never done, and who decides
## Before finishing   a short checklist, so nothing is declared done early
```

Don't add a section to fill it. A skill with no configuration has no Config
section; empty headings cost tokens and teach the reader to skim.

- Keep `SKILL.md` under 500 lines. Past that, split into sibling files and link
  them from `SKILL.md`.
- **References one level deep.** Every extra file links directly from `SKILL.md`,
  never file → file → file; nested references get partially read.
- Give reference files longer than 100 lines a table of contents at the top.
- Name files for their content: `form-validation-rules.md`, not `doc2.md`.
- Forward slashes in every path, always.
- Say explicitly whether a bundled script is to be **run** ("Run `analyze.py` to
  extract fields") or **read** ("See `analyze.py` for the algorithm").

## Guardrails

- **No time-sensitive statements.** Not "before August, use the old API". Put
  superseded material under an "Old patterns" heading instead.
- **One term per concept**, throughout. Not "field", "box", "element" for the
  same thing.
- Name MCP tools fully qualified: `ServerName:tool_name`.
- Don't assume a package is installed; say how to get it.
- In scripts: handle the error rather than deferring it to the agent, and justify
  every constant. `TIMEOUT = 47` with no reason is a bug in the instructions.

## This repo's conventions

- Skills live in `skills/<name>/SKILL.md` here, and are symlinked into
  `~/.claude/skills/<name>`.
- **No internal data in a committed skill**: no member IDs, document links, cloud
  IDs, hostnames or colleagues' names. Real values go in `config.local.md` beside
  the `SKILL.md` — gitignored — and the `SKILL.md` documents the shape and tells
  the agent to ask for what's missing.
- Write the skill for someone other than the author. "The user", not a first name.
- Add a row to the README table when adding a skill here.

## Updating a skill

Update from an observed failure, not a hunch. Ask what the agent did wrong with
the skill loaded, then change the thing that would have prevented it: make the
rule more prominent, state it in stronger terms, or move it into the workflow
step where it gets skipped.

Prefer rewriting over appending. A skill that grows by accretion stops being read.
If a rule turned out to be specific to one project or one task, abstract it or
drop it — the body should carry only what generalizes.

## Before finishing

- [ ] Description is third person, says what *and* when, and lists trigger phrasings
- [ ] `name` is lowercase-hyphen, under 64 chars, no reserved words
- [ ] Body under 500 lines; extra files linked one level deep
- [ ] Nothing in it that the model already knows
- [ ] Concrete examples, one default per decision
- [ ] No dates, no deadlines, no "for now"
- [ ] Consistent terminology
- [ ] No internal identifiers, links or names
- [ ] Tell the user how to try it, and offer to test it on a real task rather
      than declaring it done
