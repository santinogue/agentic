---
name: writing-pull-requests
description: Opens and maintains pull requests — branch checks, PR size, the What/Why body format, stacking oversized work, and keeping titles and bodies current as the branch changes. Use when the user asks to open, create, update or fix a PR, says "abrí un PR", "subí esto", "actualizá la descripción del PR", or when work on a branch is finished and needs to reach review.
---

# Writing pull requests

Gets a branch to review in a shape the reviewer can act on: right-sized, with a
title and body that still describe the diff after the last push. Adapted from
[nathancoleman/agentic-nate](https://github.com/nathancoleman/agentic-nate).

For responding to review comments, this is the wrong skill.

## Body format

Every PR body uses exactly this:

```
## What
- the change, in one or two bullets

## Why
- the reason / ticket link
```

- **Title**: imperative mood, under 70 characters. "Scale ECS clusters to 0",
  not "Scaling ECS clusters" or "This PR scales ECS clusters".
- **Language**: English, title and body.
- `## What` and `## Why` are the only required sections. Do not add Testing,
  Rollback, Screenshots or similar unless the user asks for them.
- If something important is unresolved — a risk, a caveat, something left out —
  add a short `## Notes` section at the end rather than padding the other two.
- The ticket link goes under `## Why`, not at the top of the body.
- Write the What as the theme of the change, not a file-by-file changelog.
  Enumerate an individual change only when it is surprising or carries risk.

Keep whatever commit and PR attribution the session is configured to add; do not
strip it.

## Before opening

- **Never open a PR from the default branch.** Create a branch first.
- Confirm the base branch is the one intended, not just whatever `gh` defaults to.
- Review the full branch delta, not only the latest commit.
- Run the checks the repo has — tests, lint, type checks, build — when they are
  proportionate to the change. If any were skipped or blocked, say so explicitly
  with the reason and what it leaves unverified.

Use `gh` for every PR operation (`gh pr create`, `gh pr edit`, `gh pr view`,
`gh pr status`) and rely on its existing auth; add no separate auth step unless
`gh` itself reports an error.

## Size

- Aim for under 400 changed lines. Slightly over is fine when the change stays
  easy to review.
- When several implementations are valid, prefer the one with the smaller net
  diff that still fully addresses the request.
- Each commit should be one logical step.
- When a change runs well past that, split it into a stack of targeted PRs, each
  a clear step in the story rather than an arbitrary slice.

## Stacked PRs

Build the stack with the `gh stack` extension rather than chaining branches and
base refs by hand:

- `gh stack init <branch1> <branch2> ...` — adopts existing branches or creates
  the missing ones, each based on the previous.
- `gh stack add <branch>` — adds one more branch on top as work continues.
- `gh stack submit` — pushes every branch and creates or updates all the PRs and
  the stack object in one step. Interactively it opens an editor for each new
  PR's title, description and draft state; default new PRs to draft. Non
  interactively, or with `--auto`, new PRs are created as drafts unless `--open`
  is passed.
- `gh stack sync` — brings branches and PR state up to date after changes.
- `gh stack view` — the stack's current position, for the closing summary.

Once submitted, GitHub renders the stack relationship on each PR, so the body
needs no "part of a stack, review #N first" note.

Every PR in the stack follows the same body format. If the repo's PR titles carry
a ticket prefix, infer it from the earlier PRs in the stack and reuse it exactly,
bracket format included, on every later one.

## Keeping a PR current

On every push to an open PR:

- Re-read the title and body against the current diff and intent, and update them
  when they no longer match. A stale body is worse than a thin one.
- Preserve any screenshots or images in the body — markdown or `<img>` — exactly
  where they are. Do not remove, reorder or rewrite them.
- For a stack, run `gh stack sync`, and re-run `gh stack submit` when titles or
  bodies changed. Re-check the ticket prefix across the whole stack.
- Do not describe intermediate states or steps that no longer exist in the final
  diff.
- While iterating on review, prefer precise edits over broad refactors unless the
  refactor is needed for correctness, safety or maintainability.

## Guardrails

- Never open a PR from the default branch.
- Commit and push only when the user asks for it.
- Never invent validation. If the tests were not run, the body does not imply
  they were.
- Do not add sections to the body that the user did not ask for.

## Before finishing

- [ ] Branch is not the default branch, and the base is right
- [ ] Title is imperative, English, under 70 characters
- [ ] Body has `## What` and `## Why`, and nothing else unless asked
- [ ] Ticket link under `## Why`, when there is a ticket
- [ ] Skipped checks are stated, not implied
- [ ] Hand over the PR URL plus a short readiness line: scope, what was
      validated, notable risks, and stack position if applicable
