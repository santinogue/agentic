---
name: refine-estimate
description: Refine a list of tasks against the actual codebase and estimate them one at a time, keeping a live table with a "refined" column that only the user can flip to true. Use for "estimate this work", "help me estimate", "how long would this take", "refinemos estas tareas", "estimar el backlog", "break this ticket down", or whenever a rough task list from a ticket, planning doc or meeting has to become numbers someone will commit to.
---

# Refine and estimate

Turns a rough task list into estimates the team can commit to, by refining one
task at a time against the real code. The value is rarely the number: it is
discovering that two tasks are the same operation, that a rule is already
implemented, or that a "simple change" hides a decision nobody has made.

Write in the language the user is writing in.

## Ground rules

**Never estimate from the ticket alone.** Read the code first. Every claim about
what exists, what is missing, or what something costs carries a `path:line`
reference. An estimate built on assumptions is worse than no estimate, because
it gets committed to.

**The user owns `refined`.** Every task starts at `false`. Only flip one to
`true` when the user explicitly says so — never on your own initiative, never
because the discussion felt conclusive.

**Verify before asserting, and go to the source when challenged.** If the user
pushes back on a finding, open whatever settles it — the dependency's own
source, the data model definition, the configuration, the history — and answer
with evidence. If they are right, correct the recommendation plainly and move
on. If they are right about the trade-off but the risk still stands, say both.

**Do not run infrastructure, production or database commands.** When a fact
needs one, hand the user the exact command to run with `!` so its output lands
in the session, and say what answer would change the estimate.

## The loop

1. **Explore first.** Before the first table, read enough of the codebase to
   know, for each task, whether the thing already exists. Report what you found —
   the first message should already correct the ticket's assumptions.
2. **Print the table** (format below), everything at `refined: false`, with a
   total.
3. **Take feedback on one task.** The user gives a decision, a constraint, a
   correction, or a question.
4. **Update and reprint the whole table**, every turn. Never print a fragment or
   say "the rest is unchanged" — the table is the working artifact and it has to
   be copyable in full at any point.
5. **Close each task by naming what is still open**, and when the user marks it
   refined without answering, write down the default that will be implemented.
6. Repeat until everything is refined.

## The table

| # | Task | Scope / state | Days | Refined |

- **Scope** says what exists today and what has to be built, compressed to a
  line. It is the column that changes most as refinement proceeds.
- **Days** are ideal dev days including tests, as a range.
- **Refined** shows `false` / `true` unambiguously.
- Print the running total under the table, and the refined-so-far subtotal once
  some rows are closed.

**Let the task structure change.** Merging tasks, collapsing them into another,
or moving cost between them is usually worth more than adjusting hours. When
several tasks turn out to be one operation with different parameters, say so and
fold them into a single row, leaving the folded rows visible at `0` so nobody
thinks they were forgotten. When one task must build something another one
reuses, put that cost in whichever ships first and say that the delivery order
decides who pays it.

## What to look for while reading the code

The findings that move estimates are usually these:

- **It already exists** — an unused endpoint, a validation that already encodes
  the rule, a helper that already implements the pattern.
- **Several tasks are one** — different entry points performing the same
  underlying change.
- **The cost is not where the ticket thinks** — the visible surface is cheap;
  the side effects, the decision, the data change and the coordination are not.
- **Shared machinery quietly redefines the terms** — framework behavior, base
  classes or global scopes that change what words like "delete", "empty" or
  "visible" actually mean in this codebase.
- **Prerequisites on a path about to be stressed** — what is missing before
  something gets paginated, searched, or called at a new volume.
- **Duplicated logic** that the new work is about to duplicate once more.

## Separate three things that arrive mixed

- **Work**: what someone has to build. This is what gets estimated.
- **Decisions**: what a human has to choose before the work can start. Name who
  decides.
- **Findings outside scope**: real problems the exploration turned up that
  nobody asked about. Record them, never absorb them silently into an estimate,
  and never expand scope to fix them without being asked. Give them a cost so
  the user can decide.

## The numbers

- Ideal dev days, tests included, always as a range.
- Convert to calendar time only once, at the end, and state the assumption (how
  many people, what runs in parallel).
- Do not pad quietly. When a number is high because of decisions, verification,
  data changes, cross-team coordination or review rather than typing, say so
  explicitly — especially if the user points out that the code itself is a
  couple of hours of work. Be honest that the implementation share compresses
  with AI and the rest does not, and offer to split the estimate into
  "implementation" and "everything else" when scope is being negotiated.

## Closing

When every row is refined, offer the user a choice of write-up and follow the
one they pick:

- **An artifact** — three sections: the summary table, implementation detail per
  task (the decisions made during refinement, not a tutorial), and open
  questions at the end with an owner for each.
- **A decision doc** — invoke the `decision-doc` skill instead.
- **Nothing** — the table in the terminal is the deliverable.

Folded tasks get no section of their own in the write-up; say once which row
absorbed them.
