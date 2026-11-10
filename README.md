# architecture-gap-analysis

A [Claude Code](https://claude.com/claude-code) skill that compares a set of target requirements
(a PRD, epic, RFC, backlog or user stories) against an existing codebase. For each requirement it
reports, with `file:line` evidence, whether the code already satisfies it, partly does, or lacks it.

Use it to scope an epic against existing code, to see what a spec already has support for, or to
sequence a batch of related changes. It is not for greenfield work with no code, or for a single
well-understood change.

The skill is read-only: its frontmatter disallows the tools that edit files, including the
code-editing tools of the [tokensave](https://github.com/aovestdipaperino/tokensave) MCP server,
which it uses for code lookups when available.

## Install

Copy the skill into your global Claude Code skills folder:

```bash
git clone https://github.com/Zindaar/architecture-gap-analysis.git
cp -r architecture-gap-analysis/architecture-gap-analysis ~/.claude/skills/
```

Or into one project's `.claude/skills/` to scope it to that project. Claude Code picks it up on
the next session; invoke it by asking for a gap analysis, or with `/architecture-gap-analysis`.

## Contents

- `architecture-gap-analysis/SKILL.md` — the skill: its purpose, method and report format.
