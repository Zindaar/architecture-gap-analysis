# architecture-gap-analysis

A [Claude Code](https://claude.com/claude-code) skill that compares a set of target requirements
(a PRD, epic, RFC, backlog or user stories) against an existing codebase. For each requirement it
reports, with `file:line` evidence, whether the code already satisfies it, partly does, or lacks it.

Use it to scope an epic against existing code, to see what a spec already has support for, or to
sequence a batch of related changes. It is not for greenfield work with no code, or for a single
well-understood change.

The skill is read-only by instruction: it tells Claude not to write code or edit files at any
point in the analysis. Its frontmatter also withholds the editing tools it can name, including
those of the [tokensave](https://github.com/aovestdipaperino/tokensave) MCP server, but Claude
Code applies that only to the turn that invokes the skill, so the instruction is what holds
across a longer conversation.

**No extra tools are required.** When a code-graph tool is available (tokensave, an LSP index,
a tags file), the skill uses it for definitions, callers and impact. Without one, it does the
same lookups with plain text search (Claude Code's Grep, Glob and Read, or `git grep` / `rg`),
confirming each match before citing it. The evidence standard is the same either way, and the
report says which method was used.

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
