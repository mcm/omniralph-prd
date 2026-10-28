# Shared PRD methodology

Every mode (`new`, `triage`, `edit`) produces the same core artifacts using the
formats below. The mode files describe the step sequence; this file is the
reference for *what the artifacts look like* and *how they are written to
Linear*.

The PRDs are executed by **omniralph**, whose ralph loop runs an epic delegated
to the Omniralph Linear agent:
it takes the next ready sub-issue, hands it to a fresh, unattended Claude
session, runs the quality gates, commits, marks the sub-issue Done, and moves on.
When every story is done it opens the pull/merge request, then fixes failed CI
and Omniralph's own review findings on that PR. omniralph reads stories from
**Linear's own fields** — the issue identifier, title, priority, sub-issue
order, workflow state, labels and "blocks" relations — plus the checkboxes in
each description. It does not parse story IDs or metadata in titles or bodies,
so don't write any.

---

## Clarifying questions

Before writing (or revising) a PRD, ask 3–5 essential clarifying questions,
targeted at the genuine ambiguities. Don't re-ask anything the user's context or
an existing issue already answers — confirm your understanding instead.

### Use lettered options

Format questions with lettered options so the user can answer quickly with
"1A, 2C, 3B" instead of writing paragraphs. Indent the options under each
question:

```
1. What is the primary goal of this feature?
   A. Improve user onboarding experience
   B. Increase user retention
   C. Reduce support burden
   D. Other: [please specify]

2. Who is the target user?
   A. New users only
   B. Existing users only
   C. All users
   D. Admin users only

3. What is the scope?
   A. Minimal viable version
   B. Full-featured implementation
   C. Just the backend/API
   D. Just the UI
```

### Adaptive questioning

After each response, decide whether to:

- Ask follow-up questions (if answers reveal complexity),
- Ask about a new aspect (if the current area is clear), or
- Generate the PRD (if you have enough context).

Typically 2–4 rounds are needed — fewer for small, well-specified work.

### Full scope up front

Everything in the PRD is in scope for its goal, and the whole epic gets built.

- When you present a PRD, state its full scope. Don't silently trim or defer
  anything the goal needs.
- Don't label stories, requirements or criteria "future enhancement",
  "stretch", "v2", "nice to have" or similar unless the user marked them that
  way.
- If something is genuinely optional, name it and ask the user whether it's in
  or out (a lettered question works). If it's in, it's a normal story or
  requirement. If it's out, it goes under Non-Goals.

### Prerequisites and manual steps

Ask about anything the work depends on that an AI agent working in the repo
cannot do itself, unless the context already answers it:

- **Other Linear issues** that must land first (another epic, a backend change,
  a design task). These become "blocks" relations **on the epic itself**, never
  on its stories (see "Stories may only be blocked by stories in the same
  epic").
- **Steps for a person**: provisioning secrets or credentials, DNS, third-party
  console or app-store setup, data migrations run by ops, design sign-off.
  These become separate **manual** sub-issues, never acceptance criteria on an
  agent story.

### Quality gates

The quality gates are the commands omniralph's agent runs before claiming a story
complete, and that omniralph itself runs after the story to verify it (with the
repository's **Run the PRD's quality gates** setting, on by default; the
repository's own verify commands run first). The PR's CI runs only once, when
the finished epic's PR opens, so **gates weaker than CI let every story pass
and then fail at the end.** Make the gates match what CI enforces.

Using the source access you established (SKILL.md → "Accessing the target
project's source"), discover them yourself, in this order:

1. **CI config first**: `.github/workflows/*.yml`, `.gitlab-ci.yml`, or
   equivalent. Take the commands the PR pipeline runs (lint, typecheck, test,
   build).
2. **Then the scripts those commands call**: `package.json` scripts, `Makefile`,
   `pyproject.toml`, and similar. Prefer an aggregate script (e.g.
   `npm run check`) when CI uses one.

This works the same whether you read those files from a local checkout or fetch
them through a git host connector (GitHub, GitLab, Bitbucket, etc.). If you can
determine them, document them without asking. If you have no source access,
ask:

```
What quality commands must pass for each user story?
   A. pnpm typecheck && pnpm lint
   B. npm run typecheck && npm run lint
   C. bun run typecheck && bun run lint
   D. Other: [specify your commands]

For UI stories, should the agent verify in a browser?
   A. Yes — the harness host has a browser tool (e.g. Playwright MCP); name it
   B. No, automated tests are sufficient
```

Don't assume browser verification is available: omniralph runs stories
unattended, so only ask for it if the user confirms the harness host has a
browser tool.

Every gate command must be non-interactive (no watch mode, no prompts) and
runnable from the repository root in a fresh checkout.

---

## PRD format

Generate the PRD with these sections. This becomes the description of the epic
(or the single issue) in Linear, and omniralph includes the epic's full
description in every story's prompt. The numbers below are for this reference
only — write each heading as a plain `##` heading (`## Goals`, not
`## 2. Goals`).

### 1. Introduction/Overview
Brief description of the feature and the problem it solves.

### 2. Goals
Specific, measurable objectives (bullet list).

### 3. Quality Gates
**CRITICAL.** omniralph finds the gates by exact format, so follow it exactly:

- The heading is literally `## Quality Gates`.
- Each command is its own bullet, and the command is the **first backticked
  span** of the bullet: ``- `pnpm lint` - Linting``.
- Anything that is not a shell command must **not** start with backticks — a
  bullet like ``- `dev-browser` skill`` would be run as a command. Write
  instructions as plain prose bullets instead.

```
## Quality Gates

These commands must pass for every user story:
- `pnpm typecheck` - Type checking
- `pnpm lint` - Linting
- `pnpm test` - Unit tests

For UI stories, also:
- Verify the change in a browser using the Playwright MCP
```

### 4. Non-Goals (Out of Scope)
What this feature will NOT include. Critical for managing scope. Only list
what the user agreed is out: this section is not a place to park work the goal
needs.

### 5. Design Considerations (Optional)
UI/UX requirements, links to mockups if available, relevant existing components
to reuse. Images and files attached in Linear are downloaded and shown to the
agent: attach anything every story needs to the epic, and anything only one
story needs to that story's sub-issue.

### 6. Technical Considerations (Optional)
Known constraints or dependencies, integration points with existing systems,
performance requirements.

### 7. Success Metrics
How you will know the feature succeeded (bullet list). Specific and measurable.

### Routing and branch lines

Omniralph routes an issue to a repository by its **team, project or label**,
configured per repository in Omniralph. Expect that routing to be in place, and
put the issue in the team or project (or give it the label) that routes it.

- **Repository tag (only when asked).** Don't add a `[repo=…]` tag unless the
  user asks for one. A tag overrides the routing: an issue tagged with a
  repository that doesn't have ralph enabled isn't routed at all. When asked,
  add `[repo=<full path>]` on its own line, with the repository's full path
  (`acme/app`, `group/subgroup/project`). One repository per tag, and no
  branch in it.
- **Branch (optional).** If the user wants a specific branch name, add
  `` > Branch: `name` ``. Otherwise omniralph uses Linear's branch name for the
  issue.

---

## T-shirt sizing and the decomposition rule

After drafting the PRD, estimate the size of the **entire** body of work — the
total effort to deliver every goal and functional requirement, not any single
piece. Weigh the number of distinct functional requirements, the breadth of
surface area touched (database, API, UI, integrations, infra), unknowns and
risk, and roughly how many focused AI-agent sessions the work would realistically
take.

The scale, smallest to largest:

- **XS** — Extra Small
- **S** — Small
- **M** — Medium
- **L** — Large
- **XL** — Extra Large

State the size you chose with a one-to-two-sentence rationale, then apply the
rule:

- **L or larger (L, XL) → decompose into an epic with sub-issues.** Continue to
  the User Stories step and create a parent issue with one sub-issue per user
  story, plus a final functional-requirements validation sub-issue.
- **Smaller than L (XS, S, M) → do NOT decompose.** Skip the User Stories step
  entirely and capture everything in a single self-contained issue, with
  acceptance criteria included directly in its description.

Tell the user the size and the resulting structure before continuing.

---

## User stories (only when L or larger)

> Perform this step only if the size is **L or larger**. If smaller, skip
> straight to Functional Requirements — the work ships as a single issue with no
> sub-issues, and its acceptance criteria belong directly in that issue's
> description.

Decompose the work into user stories. Each story becomes one sub-issue and is
implemented in one fresh, unattended AI-agent session, so it must be small
enough to finish in one session. Each needs a title, a description ("As a
[user], I want [feature] so that [benefit]"), and a verifiable
acceptance-criteria checklist.

**Split by feature, not by phase.** Each story is a slice of the feature that
works end to end: "Auth system" is one story, not four stories for scaffolding,
logic, tests and polish. If a feature is too big for one session, split it
into smaller features (e.g. "Sign up with email", "Password reset"), each one
complete on its own.

- A story's tests, error handling and basic hardening (input validation,
  auth checks, safe failure) belong in the story that introduces the code.
  Write them as acceptance criteria there, never as a separate story.
- A story never leaves stubs, placeholders or TODOs for anything in its own
  acceptance criteria, and never relies on a later story to finish its code.
  If part of the work can't be finished in the story, it's a different
  feature: make it its own story, wired with a blocks relation.

**Drafting labels.** While drafting with the user, refer to stories as US-001,
US-002, … so dependencies are easy to discuss. These labels are for the
conversation only: they never go into Linear titles or bodies. Once the
sub-issues exist, their Linear identifiers (e.g. `ENG-12`) are the story names —
omniralph uses them in plans, prompts and commit messages
(`feat(ENG-12): Add priority field to database`).

**Present each story as:**

```
US-001 — Add priority field to database
Priority: High · Blocked by: — · Manual: no

As a developer, I want to store task priority in the database so that it
persists across sessions.

Acceptance criteria:
- [ ] ...
```

**Sub-issue title:** the plain story title. No `US-001:` prefix.

**Sub-issue description** — exactly this shape, nothing else:

```markdown
As a [user], I want [feature] so that [benefit].

[Optional extra context: files to touch, patterns to follow.]

## Acceptance Criteria

- [ ] Specific verifiable criterion
- [ ] Another criterion
```

- Don't repeat the title as a heading, and don't add a `## Ralph Metadata`
  section, Story ID, priority or dependency lines. Priority, order and
  dependencies are set as Linear fields.
- The heading is exactly `## Acceptance Criteria` — no trailing colon, or
  omniralph won't recognize the section.

**Order, priority and dependencies.** omniralph picks the next story that has no
open blockers: in-progress stories first, then by **Linear priority** (Urgent
first, *No priority* last), then by the sub-issues' order in the parent.

- Use **blocks relations** for every real dependency. They are the only thing
  that stops a story from running early, and they alone must make the epic
  run correctly. If a story would break when run before another, that's a
  missing blocks relation — add it.
- **Stories may only be blocked by stories in the same epic.** A story's
  `blockedBy` list must contain sub-issues of its own epic and nothing else —
  never an issue outside the epic (another epic, a story in another epic, a
  standalone issue, a backend or design task). This is a hard rule with no
  exceptions. If any story is functionally blocked by an outside issue, set
  that issue on the **epic's** `blockedBy` instead, even if only one story
  needs it; omniralph then waits for it before starting any story. Manual
  steps for a person are sub-issues of the epic, so stories may depend on
  them.
- Give every story a priority. Use one priority for the whole epic unless some
  stories genuinely matter more.
- Sub-issue order in the parent is cosmetic: it only breaks ties between
  stories that are equally ready, which by definition can run in either order.
  Create sub-issues in story order, but never check, report or ask about where
  they ended up in the parent — don't tell the user to drag or reorder them,
  and don't remark that one landed out of place.

**Manual stories.** A step for a person (see "Prerequisites and manual steps")
is its own sub-issue with the **manual** label (or the label `LINEAR.md`
names), wired with blocks relations like any other story. omniralph never works
it; stories that depend on it wait until a person moves it to Done.

**Guidelines:**

- Acceptance criteria must be verifiable, not vague. "Works correctly" is bad;
  "Button shows confirmation dialog before deleting" is good.
- The agent must be able to check every criterion itself, inside the repo:
  nothing that needs human judgment, production access or an external account.
  A story whose criteria the agent can never meet is retried, then set aside,
  and every story after it is blocked.
- Put tests and any docs a story needs in that story's acceptance criteria:
  no separate testing, hardening or documentation stories.

---

## Functional requirements

Create a numbered list of specific functionalities:

- "FR-1: The system must allow users to..."
- "FR-2: When a user clicks X, the system must..."

Be explicit and unambiguous — these determine the final success of the
implementation regardless of size. Write them as bullets, never as headings.
If the PRD is **L or larger**, they become the final functional-requirements
validation sub-issue of the epic. If smaller, they live directly in the single
issue's description alongside its acceptance criteria.

### The functional-requirements validation sub-issue

In an epic, this is the last story, blocked by every other sub-issue. By
default it is an **agent story**: the agent checks each FR against the code,
adds missing tests and fixes gaps. A pass that changes nothing is fine. Human
review happens on the PR, which omniralph opens once this story is done.

Title: `Validate functional requirements`. Description:

```markdown
Verify that the finished epic meets every functional requirement and every
story's acceptance criteria. For each item, confirm it in the code or with a
test; add missing tests and fix any gap you find.

Stories covered: ENG-12, ENG-13, ENG-14

## Acceptance Criteria

- [ ] FR-1: ...
- [ ] FR-2: ...
- [ ] All acceptance criteria of the stories above are met
- [ ] All quality gates pass
```

Make it a **manual** story only if the user explicitly wants a person to sign
off before the PR exists. Warn them first: omniralph opens the PR only after
every story is done, so a manual validation story means no PR at all until the
person closes it — unless the repository sets `openPullRequestEarly`.

---

## Writing quality

The PRD will be executed by AI coding agents via omniralph, one story per fresh
session. Therefore:

- Be explicit and unambiguous.
- User stories should be small (completable in one session), each a complete
  feature slice that includes its own tests and error handling.
- Acceptance criteria must be machine-verifiable, by the agent, in the repo.
- Include specific file paths if you know them.
- Reference existing code patterns in the project.

---

## Linear structure reference

When you write to Linear (the exact create-vs-update actions are in each mode
file), the target structure is one of:

**Epic (L or larger):**

- A parent/epic issue whose description is the PRD body, including the branch
  line (and a repository tag, if the user asked for one).
- One sub-issue per user story, created **in story order**, each with:
  - `parentId` = the epic, `title` = the plain story title, `description` = the
    story body in the format above;
  - `priority` (1 Urgent, 2 High, 3 Medium, 4 Low) — always set;
  - `blockedBy` = only the sibling sub-issues of this epic it depends on —
    **never** an issue outside the epic (see "Stories may only be blocked by
    stories in the same epic");
  - the **manual** label (`addLabels`) for steps for a person.
- One final functional-requirements validation sub-issue (see above),
  `blockedBy` every other sub-issue.
- Every dependency on an issue outside the epic — whether the whole epic or
  just one story needs it — goes on the **epic's** `blockedBy`: omniralph
  won't start any story until it's done.

After creating them:

1. Replace drafting labels with Linear identifiers wherever they ended up in a
   description (story cross-references, the validation sub-issue's "Stories
   covered" line).
2. Verify every dependency you drafted exists as a blocks relation, and that
   no sub-issue is blocked by anything outside the epic. If one is, remove that
   relation from the sub-issue and add it to the epic instead.

**Single issue (smaller than L):**

- One issue whose description is the PRD body and branch line, then the
  functional requirements, then a consolidated acceptance-criteria checklist
  under `## Acceptance Criteria` as the **last** section. No sub-issues.
- omniralph treats the whole description as one story, and would split it into
  sub-issues if it looked like a PRD with stories. So in a single issue:
  - no story-style headings such as `### US-001: …`, `### FR-1: …` or
    `### Feature 1.1: …` — keep FRs and criteria as bullets and checkboxes;
  - no ```` ```json ```` block containing `userStories`;
  - no `## Description` or `## Test Plan` headings.
- Set `blockedBy` for any issue it waits on, as for an epic.

Whenever removing a story/issue in Linear, **cancel** it (do not delete). Never
modify a sub-issue that is already completed or in progress, and never
restructure an issue, without explicit user confirmation.

### Delegating to omniralph

Delegating is what starts the run, and it's the user's call. Never offer to
delegate, and don't suggest it as a next step. Only if the user asks, delegate
the epic (or single issue) with `save_issue` and `delegate`, using the agent
name from `LINEAR.md`, else `Omniralph`.

---

## Worked example

### PRD

```markdown
# PRD: Task Priority System

## Introduction

Add priority levels to tasks so users can focus on what matters most. Tasks can
be marked as high, medium, or low priority, with visual indicators and
filtering to help users manage their workload effectively.

## Goals

- Allow assigning priority (high/medium/low) to any task
- Provide clear visual differentiation between priority levels
- Enable filtering and sorting by priority
- Default new tasks to medium priority

## Quality Gates

These commands must pass for every user story:
- `pnpm typecheck` - Type checking
- `pnpm lint` - Linting
- `pnpm test` - Unit tests

## Non-Goals

- No priority-based notifications or reminders
- No automatic priority assignment based on due date
- No priority inheritance for subtasks

## Technical Considerations

- Reuse existing badge component with color variants
- Filter state managed via URL search params
- Priority stored in database, not computed

## Success Metrics

- Users can change priority in under 2 clicks
- High-priority tasks immediately visible at top of lists
- No regression in task list performance
```

### User Stories

Sub-issue 1 — title `Add priority field to database`, priority High, no
blockers:

```markdown
As a developer, I want to store task priority in the database so that it
persists across sessions.

## Acceptance Criteria

- [ ] Add priority column to tasks table: 'high' | 'medium' | 'low' (default 'medium')
- [ ] Generate and run migration successfully
- [ ] Saving a task with a value outside 'high' | 'medium' | 'low' is rejected
- [ ] Tests cover the default value and the rejected value
- [ ] Typecheck passes
```

Sub-issue 2 — title `Display priority indicator on task cards`, priority High,
blocked by sub-issue 1:

```markdown
As a user, I want to see task priority at a glance so that I know what needs
attention first.

## Acceptance Criteria

- [ ] Each task card shows colored priority badge (red=high, yellow=medium, gray=low)
- [ ] Priority visible without hovering or clicking
- [ ] Component test covers the badge for each priority level
- [ ] Typecheck passes
```

### Functional Requirements

```markdown
- FR-1: Add `priority` field to tasks table ('high' | 'medium' | 'low', default 'medium')
- FR-2: Display colored priority badge on each task card
- FR-3: Include priority selector in task edit modal
- FR-4: Add priority filter dropdown to task list header
- FR-5: Sort by priority within each status column (high > medium > low)
```
