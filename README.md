# agentic

Personal playbooks for AI coding agents: reusable procedures I'd otherwise have
to re-explain every session. Written for [Claude
Code](https://claude.com/claude-code) skills, but they're plain Markdown — the
process in each one is portable to any agent that can follow written
instructions.

## Skills

| Skill | What it does |
|---|---|
| [`refining-estimates`](skills/refining-estimates) | Refines a rough task list against the real codebase and estimates it one task at a time, keeping a live table with a `refined` column only the user can flip. |
| [`writing-decision-docs`](skills/writing-decision-docs) | Writes an analysis in a decision-first format — *In one minute, Decisions to be made, Recommendations, How we got here, Appendix* — and publishes it as a Google Doc. |
| [`reviewing-jira-boards`](skills/reviewing-jira-boards) | Reviews a Jira board for project-management health (hygiene, stuck work, unanswered comments, whether the ready queue covers the next two weeks) and sends the summary as a Slack DM. |
| [`reviewing-pull-requests`](skills/reviewing-pull-requests) | Walks a PR file by file at your pace, drafts inline comments, confirms each one before it exists, and publishes them as a single review in your name. |
| [`writing-pull-requests`](skills/writing-pull-requests) | Opens and maintains PRs: branch checks, size limits, the `## What` / `## Why` body format, stacking oversized work, and keeping the body current as the branch changes. |
| [`writing-skills`](skills/writing-skills) | Creates or updates a skill in this repo, following Anthropic's authoring best practices and the conventions below. |

## Install

Skills are picked up from `~/.claude/skills/<name>/SKILL.md`. Symlink the ones
you want:

```bash
git clone https://github.com/santinogue/agentic.git
ln -s "$PWD/agentic/skills/refining-estimates" ~/.claude/skills/refining-estimates
```

Invoke one by name (`/refining-estimates`) or just describe the task — the
`description` in the frontmatter is what makes a skill fire on its own.

## Configuration and secrets

Nothing in this repo holds an ID, a URL or a name that isn't mine to publish.
A skill that needs real values reads them from a `config.local.md` sitting next
to its `SKILL.md`, which is gitignored; the `SKILL.md` documents the shape and
the agent asks for anything missing.

Keep it that way when adding a skill: board keys, member IDs, document links and
colleagues' names belong in the local file, not in the committed one.

## Writing a skill

What makes these work, beyond the content:

- **The `description` is the trigger.** Write it for matching, not for
  elegance — say what the skill does and list the phrasings that should fire it,
  including the ones in the language you actually type in.
- **Instructions, not documentation.** Imperative, addressed to the agent.
  Describe the process, not the topic.
- **Encode the judgment, not just the steps.** The reusable part is usually a
  rule the agent would otherwise get wrong: what not to do, who decides, when to
  stop and ask.
- **Keep it tool-agnostic where you can.** Name a specific tool only where the
  skill genuinely depends on it.
- **Name skills in gerund form** — `refining-estimates`, not `refine-estimate` —
  so the collection stays consistent.
- **Share a spine, not a template.** Every skill opens with one paragraph saying
  what it produces, and closes with `## Guardrails` and `## Before finishing`.
  The middle takes whatever shape the task needs — no empty sections.

The [`writing-skills`](skills/writing-skills) skill encodes all of this: ask for
it when adding or changing one.
