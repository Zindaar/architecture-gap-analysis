---
name: architecture-gap-analysis
description: "Compares a set of target requirements — a PRD, epic, RFC, backlog, or user stories — against an existing codebase and reports, with file:line evidence, which are already satisfied, which are partial, and which are absent. Use when asked what it would take to build a set of features on top of the current system, to scope an epic against existing code, to check what a spec already has support for, or to sequence a batch of related changes against a real repository. Not for greenfield work with no existing code, and not for a single well-understood change."
disallowed-tools:
  - Write
  - Edit
  - NotebookEdit
  - mcp__tokensave__tokensave_str_replace
  - mcp__tokensave__tokensave_multi_str_replace
  - mcp__tokensave__tokensave_replace_symbol
  - mcp__tokensave__tokensave_insert_at
  - mcp__tokensave__tokensave_insert_at_symbol
  - mcp__tokensave__tokensave_delete_symbol
  - mcp__tokensave__tokensave_replace_lines
  - mcp__tokensave__tokensave_rename
  - mcp__tokensave__tokensave_record_decision
  - mcp__tokensave__tokensave_record_code_area
metadata:
  category: analysis
  tags:
    - architecture
    - gap-analysis
    - planning
    - requirements
    - code-audit
  date_added: "2026-09-03"
---

# Architecture Gap Analysis

## Purpose

To determine, with citable evidence, which of a set of target requirements the current
codebase already satisfies, which it partly satisfies, and which are absent — then to
name the architectural changes required and sequence the work so that each step leaves
the system in a working state.

The output is an analysis, not an implementation. Do not write production code while
running this skill. That holds whatever tools this session happens to expose, and whether
or not any of them are withheld — do not edit through a shell, and treat the mutating tools
of any code-graph or refactoring server as equally out of bounds. The frontmatter withholds
the editing tools it can name as a second line of defence, not as the reason.

## When to use

- A PRD, epic, backlog, RFC, or set of user stories must be assessed against an existing system.
- Someone asks what it would take to build a set of features on top of what already exists.
- Scoping or estimating a body of work before committing to it.
- Deciding the order of execution for a batch of related changes against a real repository.

## When NOT to use

- **A single well-understood change.** Just make it.
- **Greenfield work with no existing code.** There is no gap to analyse; write a design instead.
- **A narrow question answerable by reading one file.** Read the file.
- **A general code-quality audit.** This skill assesses debt only where it collides with the
  target requirements. An audit with no spec attached is a different task.

## Step 1 — Resolve both sides. Do not open by asking for a paste.

Two inputs are required. Obtain them; do not request them as pasted blobs.

**The requirements.** Accept a file path, an issue or ticket reference, a URL, a directory
of specs, or text the user has already supplied. Accepting text already offered is fine —
the rule forbids *demanding* a paste, not reading one. Resolve an issue reference with
whatever this session provides (a `gh`/`jira` CLI, a tracker MCP server, an authenticated
fetch); if none can reach it, say so and ask for the text. If no source is named at all,
ask once for the location and wait for the answer before continuing.

**The codebase.** Default to the current working directory's repository. If the requirements
plainly target a different repository or subdirectory, say which one is being analysed
before starting, so a wrong target costs one line rather than a full sweep.

**Record the ref.** Capture the branch and commit being analysed with
`git rev-parse --abbrev-ref HEAD` and `git rev-parse HEAD`. Do not read `.git/HEAD` and
chase the ref file by hand — that breaks on packed refs, submodules, and worktrees. If the
ref genuinely cannot be determined (not a git checkout, no shell), say so explicitly in the
preamble rather than omitting it. Every verdict is only true of a specific tree.

If either side cannot be resolved, stop and say which. An analysis against an invented
codebase or invented requirements is worse than no analysis.

## Step 2 — Atomize the requirements

Prose requirements bundle several testable claims into one sentence. Split them.

Convert the source into a numbered list of **atomic, independently verifiable**
requirements: `R1`, `R2`, … Each must be assessable as satisfied or not on its own. Carry
the originating wording with each ID so every row traces back to the spec.

> "As an admin I can invite users by email and set their role"
>
> - **R1** Invite a user by email — *("As an admin I can invite users by email…")*
> - **R2** Assign a role at invite time — *("…and set their role")*
> - *Candidate, not an `R`*: restrict both to admins — *("As an admin…")*. The story names
>   who acts, not that others are barred. Unless the spec states the restriction, this is an
>   interpretation: it goes to Open Questions, and it gets no ID.

Anything too vague to verify — or verifiable but never actually stated — does **not**
become an `R`. Collect those for Open Questions —
an unverifiable requirement silently assessed against an invented interpretation is the
most common way this analysis goes wrong.

**Batching.** If atomization yields more than 25 requirements, group them by feature area
into batches of at most 15 and run **Step 3** per batch. Emit **one consolidated** output
covering every batch — never one report per batch — and name the batch boundaries in the
preamble. Batching changes the working order, not the output contract or the evidence
standard.

## Step 3 — Sweep for evidence, per requirement

For each `R`, search for its implementation. Search by concept, not only by the
requirement's vocabulary — the code will use different names than the spec. Look for the
domain nouns, the persistence layer, the API surface, and the entry points.

If this session exposes a code-graph tool — a symbol-index MCP server, an LSP index, a tags
file — prefer it for symbol lookup, callers, and impact; it is faster and cheaper than text
search. Otherwise use ordinary search and file reads. The evidence standard below is
identical either way, and no finding may rest on a tool's summary that was not confirmed
against the cited line.

**Evidence rules — these are what make the analysis trustworthy:**

1. Every `Fully` or `Partially` verdict cites at least one `path/to/file.ext:line`. If no
   citation can be produced, the verdict is not yet earned — **search again**. Once a
   negative search is documented, record `Missing` per rule 2; that is the exit. Never
   downgrade an unsearched requirement to `Missing`, and never route a thin search into
   `Unknown`; `Unknown` is reserved for rule 3.
2. `Missing` requires a *negative* search: state which terms, symbols, or directories were
   checked and came back empty. "I did not find it" is a finding only once the looking is
   documented.
3. Distinguish **absent** from **unsearchable**. If a subsystem genuinely cannot be
   searched — generated code, an unchecked-out submodule, a binary dependency, an external
   service — mark the requirement `Unknown` and name that blocker. `Unknown` always has a
   blocker; if there is none, rule 1 applies instead.
4. Never infer capability from a name. A `PermissionService` class does not establish that
   permissions are enforced; the call site that enforces them does.
5. Configuration, migrations, and tests are evidence too — a migration that adds the column
   is stronger evidence of a data model than a DTO that mentions it.
6. **A `Partially` verdict makes two claims and both need evidence**: a citation for the
   part that exists, *and* a documented search for the part declared absent. An absence
   asserted from structure alone is not established — go and look.

## Step 4 — Output

Emit a short preamble followed by **five** numbered sections, in this order.
Output is chat-only. Do not write a report file during the analysis; if one is wanted,
offer it once the analysis has ended.

**Preamble.** What was analysed — repo, branch and commit, requirement source, and batch
boundaries if batched — then the headline count: e.g. "18 requirements: 5 fully supported,
6 partial, 5 missing, 2 unknown."

### 1. Feature Coverage Matrix

One row per `R`, in ID order. Example row — illustrative only, never echo these values:

| ID | Requirement | Status | Evidence | Confidence |
|----|-------------|--------|----------|------------|
| R1 | Invite a user by email — *("As an admin I can invite users by email…")* | Partially Supported | `src/users/invite.ts:44` sends the invite; no expiry — searched `expir`, `ttl`, `revoke` across `src/users/`, `migrations/`: none | High |

Status rubric — apply strictly:

| Status | Means |
|---|---|
| Fully Supported | Behaviour exists end to end and is reachable from a real entry point. |
| Partially Supported | Some layer exists but a named piece is absent — no API exposure, no persistence, no authorization, no error path. Name the piece. |
| Missing | Documented negative search; no implementation found. |
| Unknown | Blocked from assessment. Name the blocker. |

Confidence rubric — for `Partially`, confidence rates the **absence** claim, since that is
the half that drives the work:

| Confidence | Means |
|---|---|
| High | `Fully`: the implementing code was read at the cited line, entry-point chain included. `Partially`: the cited line was read **and** the absent piece was searched for across the plausible names and layers, with terms stated. `Missing`: a broad negative search across the plausible names, layers, and directories. |
| Medium | A signature, call site, migration, or config entry was seen; the body was not read. `Fully`: admissible only when the entry-point chain itself was read. `Partially`: the cited line was read and the absence was searched for, but narrowly — one obvious location, or one term family. `Missing`: a competent but narrow search. |
| Low | Inferred from adjacent code, naming, or documentation. `Fully`: not admissible — go and read the implementation. `Partially`: not admissible — read the implementation, then run the rule-6 absence search before recording `Partially`. `Missing`: a single-term or single-directory search only; say so, and widen it if the requirement matters. |

`Unknown` rows carry no confidence — use `—`.

### 2. Architectural & Schema Gaps

Grouped by layer — data model, API surface, domain/service layer, state management,
integration, infrastructure. For each gap: what must change, which `R` IDs depend on it,
and whether the change is **additive** (backward compatible) or **breaking**. Both the IDs
and the additive/breaking label are required on every gap.

Call out explicitly every change that is hard to reverse: destructive migrations, data
backfills, public API contract changes, and anything touching auth or tenancy.

### 3. Technical Debt & Risks

Only debt that **collides with these requirements**. Every entry must name the `R` IDs it
obstructs; an entry that cannot is out of scope and belongs in a separate audit.

Each entry: the pattern, where it lives (`file:line`), which requirements it obstructs, and
the consequence of leaving it. Include security and performance concerns *when the new
requirements touch them* — a new bulk endpoint over an N+1 query is in scope; an unrelated
N+1 elsewhere is not.

Distinguish observed facts from suspicions. Label an unverified concern as such rather than
promoting it to a finding.

### 4. Step-by-Step Implementation Plan

Ordered steps. Prefer dependency order over layer dogma — "database first, frontend last"
is a useful default, not a rule; a vertical slice is often the better first step because it
proves the design end to end.

For each step: what changes, which `R` IDs it advances, what it depends on, relative size,
and how it is verified. Do not invent hour or day estimates. Sizes are anchored as:

- **S** — confined to one file or module; no contract change.
- **M** — spans several modules, or adds an endpoint, table, or column; backward compatible.
- **L** — changes or breaks a public contract, crosses a trust boundary, or needs a destructive migration, a
  backfill, or a dual-write window. Irreversibility is what makes a step `L`; an additive
  migration alone is `M`.

Mark every point at which the system would be left broken or migration-locked, and state
how to avoid it: expand-contract migration, feature flag, dual-write, compatibility shim.

### 5. Open Questions

Mandatory; never omitted and never empty without justification. Contains the requirements
set aside in Step 2, every `Unknown` row with its blocker, and every
assumption the plan rests on. If any plan step depends on an `Unknown`, say plainly that
the plan is provisional past that point.

## Step 5 — Self-check before emitting

Not an output section. Verify each item against the drafted text, then emit:

- All five sections are present, plus the preamble.
- The preamble states the repo, the branch and commit — or why the ref could not be
  determined — and its counts match the matrix.
- Every `R` from Step 2 has exactly one matrix row, and no row bears an ID Step 2 did not produce.
- Every `Fully` and `Partially` row carries a `file:line` citation.
- Every `Missing` row, and the absence half of every `Partially` row, names the terms or
  directories searched.
- No `Fully` or `Partially` row carries `Low` confidence; every `Unknown` row carries `—`.
- Every `Unknown` row names a blocker and appears in Open Questions.
- Every Section-2 gap names its dependent `R` IDs and is labelled additive or breaking, and
  the hard-to-reverse changes are called out as such.
- Every Section-3 debt entry names the `R` IDs it obstructs.
- Every Section-4 step names the `R` IDs it advances, carries a size and a verification
  method, and — where marked as leaving the system broken or migration-locked — names its
  avoidance strategy.
- If batched: the output is consolidated and covers every batch.

Checked against the Step 2 working list rather than the draft, since the draft is the only
other record: every vague or unstated requirement set aside in Step 2 appears in Open
Questions.

## Failure modes to avoid

- **Reporting `Missing` for code that exists** under a different name. Search by concept.
- **Padding the matrix** so every requirement earns a paragraph. A three-word evidence
  clause that cites a line beats prose that cites nothing.
- **Scope drift into a full audit.** Every finding traces to a requirement ID.
- **Silent interpretation** of a vague requirement. It goes to Open Questions instead.
- **Asserting an absence without looking**, then rating it High.
- **Confident sequencing over unknowns.** Say where the plan turns provisional.
- **Writing code.** This skill ends at the plan.

## Limitations

- The analysis reflects the tree as read; the preamble states the branch and commit, or
  says why it could not.
- Verdicts derive from static reading, not execution. Runtime behaviour, infrastructure
  config, and external services are asserted only where evidence was actually seen.
- Not a substitute for review by someone who knows the system's history.
