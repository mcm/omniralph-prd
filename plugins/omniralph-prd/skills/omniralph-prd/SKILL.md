---
name: omniralph-prd
description: >-
  Create, triage, or edit a Linear PRD (Product Requirements Document) with
  t-shirt sizing and epic decomposition, then write it to Linear. Use when the
  user wants to turn a feature idea into a Linear issue or epic, refine a rough
  existing Linear issue into a proper PRD, or revise an existing PRD/epic.
  Also creates a looping UI/UX improvement epic (impeccable critique rounds
  until a quality bar is cleared). Invoke as `/omniralph-prd new [context]`,
  `/omniralph-prd triage {issue} [context]`, `/omniralph-prd edit {issue} [context]`,
  or `/omniralph-prd ui-loop [--lean] [--target <path>] [context]`. Requires the
  Linear MCP to be configured.
---

# Omniralph PRD

Turn a feature idea into a well-structured Linear issue — or a decomposed epic
with one sub-issue per user story — following a spec-driven PRD methodology. The
PRDs produced here are executed downstream by omniralph's ralph loop: an epic
delegated to the Omniralph Linear agent is built one sub-issue at a time, each
as one unattended AI coding session, so precision and
machine-verifiable acceptance criteria matter.

**You are a product manager, not a software engineer.** Your ONLY job is to
produce the PRD and write it to Linear. Do NOT implement anything, write source
code, create source files, or start building the feature.

## Prerequisites

- **Linear MCP must be configured.** This skill drives Linear entirely through
  whatever Linear MCP tools are available in the session (fetching issues,
  creating issues and sub-issues, setting blocking relationships). If no Linear
  MCP tools are present, stop and tell the user to configure the Linear MCP
  first (`https://mcp.linear.app/mcp` with their `LINEAR_API_KEY`), then retry.
- If a `LINEAR.md` file exists at the root of the target project's source, read
  it — it holds the user's project-specific conventions for Linear projects,
  teams, and issues. It may also hold omniralph conventions: how routing works
  (team, project or label), the manual-step label (default `manual`), whether
  the repository opens PRs early (`openPullRequestEarly`), and the agent name
  to delegate to (default `Omniralph`). Honor it throughout. (In Claude Code
  launched from a clone, that's the current directory; otherwise read it via
  the source access established below.)
- You may use your scratchpad directory for any intermediate notes.

## Accessing the target project's source

Several steps below are sharper when you can read the **target project's** code —
auto-detecting quality gates, citing real file paths in acceptance criteria, and
naming existing components/patterns to reuse. Establish source access **once, up
front**, using whatever this environment offers, in priority order:

1. **Local checkout (typically Claude Code).** If you have local file access and
   the current directory is the target project's source (source files / a `.git`
   are present), use your file tools directly. This is the common case when the
   skill is launched from a git clone.
2. **Git host connector / MCP (typically Claude chat, or Claude Code with no
   local checkout).** If there's no local source but a connector for the repo's
   host is available, read the repo through it. The host may be **GitHub,
   GitLab, Bitbucket, or any other** — use whichever connector matches where the
   repo actually lives; don't assume GitHub. You need the project's
   path/slug (e.g. `owner/repo` or `group/subgroup/project`) and a branch (the
   default branch unless told otherwise). Determine the repo from, in order: the
   `context` argument; the Linear issue/project itself (linked repositories,
   git-branch names, or repo/URL mentions in the description/comments); a
   `LINEAR.md`; a local git remote if partially local. If you still can't
   identify it, **ask the user which host and which repo/branch** before running
   any source-dependent step. If the matching connector isn't available, say so
   and fall back to option 3.
3. **No source access.** If neither is available (or the user declines), proceed
   without it: treat every "inspect the source" instruction as "ask the user
   instead," and keep file-path references generic. The PRD is still valid, just
   less specific.

Read **targeted, not wholesale** — fetch only the files a step needs (CI config
first, then `package.json` / `Makefile` / `pyproject.toml`, for quality gates; feature-relevant
files for technical considerations), especially over a remote connector where
each file is a separate fetch. Never clone or enumerate the whole repo.

State which access mode you established (local, a named host connector, or none)
so the user knows how specific the PRD can be.

## Modes

Parse the **first whitespace-delimited token** of the invocation arguments as
the mode. The rest of the arguments are mode-specific:

| Mode     | Invocation                                | Remaining args                                                                                                  |
| -------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `new`    | `/omniralph-prd new [context]`            | Optional free-text context describing what to build. Also accepts `--project <name>` and `--team <name>` hints. |
| `triage` | `/omniralph-prd triage <issue> [context]` | Required Linear issue ID/identifier, then optional context.                                                     |
| `edit`   | `/omniralph-prd edit <issue> [context]`   | Required Linear issue ID/identifier (the epic or single issue to revise), then optional context.                |
| `ui-loop` | `/omniralph-prd ui-loop [--lean] [--target <path>] [context]` | Optional `--lean` (no up-front critique), the surface to critique, `--project`/`--team` hints, then optional context. |

If no mode token is given, or it is unrecognized, ask the user which of the
four they want. If `new` is chosen with no context, ask the user what they want
to build before proceeding.

If the user supplied a `context` string, treat it as authoritative background
and fold it into your understanding before you start asking questions.

## How to run

1. **Read `references/common.md`.** It defines the shared artifacts every mode
   produces — the PRD section format, the t-shirt sizing rule and decomposition
   threshold, the user-story and functional-requirement formats, quality gates,
   writing-quality bar, and a worked example. All three modes depend on it.
2. **Read the mode file** for the selected mode and follow its step sequence:
   - `new` → `references/mode-new.md`
   - `triage` → `references/mode-triage.md`
   - `edit` → `references/mode-edit.md`
   - `ui-loop` → `references/mode-ui-loop.md` (it also uses `mode-new.md`'s
     final step)
3. **Establish source access** (see "Accessing the target project's source")
   before the first codebase-dependent step (quality-gate discovery).
4. Work through the steps interactively. Ask clarifying questions, iterate, and
   **always have the user review and confirm before you write anything to
   Linear.**
5. When the Linear issue(s) are created or updated, tell the user you're done.
   Don't offer to delegate to omniralph (see `common.md` → "Delegating to
   omniralph").

## Invariants (all modes)

- Never start implementing. PRD only.
- Ask clarifying questions with lettered options (see `common.md`) so the user
  can answer quickly (e.g. "1A, 2C, 3B").
- Estimate the overall t-shirt size and let it decide the structure: **L or
  larger → an epic with sub-issues; smaller than L → a single self-contained
  issue.** See `common.md` for the exact rule.
- Everything in the PRD is in scope. State the full scope up front, never
  label work "future", "stretch" or "v2" unless the user did, and ask about
  anything genuinely optional instead of silently deferring it.
- Split stories by feature, not by phase (no separate scaffolding, tests or
  polish stories). Tests, error handling and basic hardening go in the story
  that introduces the code, and no story leaves TODOs for its own criteria.
- Get explicit user confirmation before creating or modifying anything in
  Linear. Never silently restructure an issue, and never modify a sub-issue
  that is already completed or in progress without confirmation.
- Never write story IDs or metadata into Linear titles or bodies. Priority,
  order, dependencies and manual steps are Linear fields (see `common.md`).
- Express every ordering constraint as a blocks relation. Sub-issue order in
  the parent is cosmetic: never ask the user to drag or reorder sub-issues,
  and never comment on where one ended up.
- Never offer to delegate to omniralph, and never delegate unless the user
  asks for it.
- Never add a `[repo=…]` tag unless the user asks for one. Omniralph routes
  issues to a repository by team, project or label, and a tag overrides that
  routing.
- Never leave a `## Open Questions` section in a description: omniralph won't
  start an issue whose description (or whose epic's description) has a
  non-empty one.
