# Mode: new

You are helping create a Product Requirements Document (PRD) for a software
feature from scratch, then saving it to Linear.

Depending on the estimated size, this becomes either a structured epic (a Linear
issue with several child issues, one per user story, when the work is **L** or
larger) or a single self-contained Linear issue (when smaller).

If the invocation included `--project` and/or `--team` hints, use them when
creating the issue(s). If a `LINEAR.md` exists, honor its conventions.

Follow the artifact formats in `common.md`.

---

## Step 1: Clarifying questions

Before writing the PRD, ask 3–5 essential clarifying questions where the initial
prompt is ambiguous. Focus on:

- **Problem/Goal:** What problem does this solve? Why does it matter?
- **Core Functionality:** What are the key actions or behaviors?
- **Scope/Boundaries:** What should it NOT do?
- **Success Criteria:** How do we know it's done and working?

Use lettered options and adaptive questioning (see `common.md`). Ask about
prerequisites and manual steps (see `common.md`). Discover the quality gates
from the source you established access to (SKILL.md), CI config first, or ask.

## Step 2: Write the PRD

After incorporating the user's answers, generate the PRD using the section
format in `common.md`. This will be used as the description of the epic (or
single issue) in Linear.

## Step 3: Estimate the overall size

Apply the t-shirt sizing rule from `common.md`. State the size, the rationale,
and the resulting structure (an epic with sub-issues, or a single issue) before
continuing.

## Step 4: User stories

Only if the size is **L or larger.** Decompose the work into user stories using
the format and guidelines in `common.md`. If smaller than L, skip to Step 5.

## Step 5: Functional requirements

Produce the numbered FR list per `common.md`.

## Final step: save to Linear

Once the PRD and functional requirements (plus user stories, if **L or larger**)
are written, have the user review and confirm they're ready to save. Only after
they confirm, use the Linear MCP tools to create the issue(s), following
`common.md` → "Linear structure reference":

**If L or larger — create an epic with sub-issues:**

- Create a new parent/epic issue.
  - Give it a descriptive but concise name.
  - Use the PRD body (with the branch line, if any) as the issue
    description.
  - Set `blockedBy` for every issue outside the epic that the epic or any of
    its stories waits on.
- Create a sub-issue for each user story, in story order.
  - Use the plain story title as the sub-issue title (no story ID).
  - Use the story body as the description, following the format carefully.
  - Set its priority, its `blockedBy` dependencies (sibling sub-issues
    only, never issues outside the epic), and the manual label for steps for
    a person.
- Create the functional-requirements validation sub-issue (format in
  `common.md`), blocked by all the other sub-issues.
- Do the post-creation steps: replace drafting labels with Linear identifiers,
  and verify the blocks relations.

**If smaller than L — create a single self-contained issue (no sub-issues):**

- Create one new issue.
  - Give it a descriptive but concise name.
  - Use the PRD body (with the branch line, if any) as the description.
  - Append the functional requirements and then a consolidated
    `## Acceptance Criteria` checklist as the last section.
  - Follow the single-issue formatting rules in `common.md` so omniralph doesn't
    split it into sub-issues.
  - Set `blockedBy` for any issue it waits on.
  - Do **not** create any sub-issues.

## Checklist before creating the issue(s)

- [ ] Asked clarifying questions with lettered options
- [ ] Incorporated the user's answers into the PRD
- [ ] Estimated the overall t-shirt size and chose the structure accordingly
      (epic only if L or larger)
- [ ] If decomposed: user stories are small, atomic, and specific
- [ ] If decomposed: stories split by feature, not by phase; each includes its
      own tests, error handling and basic hardening
- [ ] Full scope stated up front; nothing labelled future/stretch/v2 unless the
      user did, and genuinely optional items were put to the user
- [ ] Acceptance criteria are verifiable (not vague)
- [ ] Functional requirements are numbered and unambiguous
- [ ] Non-goals section defines clear boundaries
- [ ] Success metrics are specific and measurable
- [ ] Quality gates match CI and follow the exact `## Quality Gates` format
- [ ] Every acceptance criterion can be checked by the agent inside the repo;
      steps for a person are separate manual sub-issues
- [ ] If decomposed: every story has a priority and its dependencies as
      blocks relations; no story IDs or metadata in titles or bodies
- [ ] If decomposed: no story is blocked by an issue outside the epic; every
      outside dependency is on the epic itself
- [ ] Issue is in the team/project (or has the label) that routes it; no
      `[repo=…]` tag unless the user asked for one
- [ ] No `## Open Questions` section left in any description

Once the issue is created, tell the user you're done.
