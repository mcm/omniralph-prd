# Mode: triage

You are helping triage an existing Linear issue into a Product Requirements
Document (PRD). The starting issue is not yet in PRD form — it may be a one-line
description, a bug report, a loose collection of thoughts, or a feature request
with partial detail. Your job is to elicit the missing context and produce the
full structure, then write it back to the **same** issue.

Depending on the estimated size, the issue becomes either a structured epic
(sub-issues for individual user stories, plus a final functional-requirements
validation sub-issue, when the work is **L** or larger) or stays a single
self-contained issue (when smaller).

Follow the artifact formats in `common.md`.

---

## Step 1: Load and summarize the issue

Before asking anything, use the Linear MCP tools to fetch the target issue.

- Read the title, description, comments, labels, and any linked issues.
- Note the issue's current state, priority, and assignee.
- Identify what is already specified vs. what is missing relative to the PRD
  format.
- If the description has a `## Open Questions` section (Omniralph's automated
  triage writes one), those questions are gaps to resolve with the user. Fold
  the answers into the PRD and drop the section: omniralph won't start the
  issue while it has open questions.

Then summarize for the user:

- What the issue says today (a brief restatement, not a copy-paste).
- What you can infer about intent.
- The specific gaps that must be filled before this can become a PRD (problem
  statement, scope, acceptance criteria, etc.).

This anchors the conversation and shows the user what's missing before you ask.

## Step 2: Clarifying questions

Ask 3–5 essential clarifying questions targeted at the gaps you identified.
Focus on:

- **Problem/Goal:** What problem does this solve? Why does it matter?
- **Core Functionality:** What are the key actions or behaviors?
- **Scope/Boundaries:** What should it NOT do?
- **Success Criteria:** How do we know it's done and working?

If the existing issue already answers some of these, don't re-ask — confirm your
understanding instead. Use lettered options and adaptive questioning (see
`common.md`). Ask about prerequisites and manual steps (see `common.md`) — the
issue's existing blocking relations may already answer this. Discover the
quality gates from the source you established access to (SKILL.md), CI config
first, or ask.

## Step 3: Write the PRD

After incorporating the user's answers, generate the PRD using the section
format in `common.md`. This will replace the current issue description and serve
as the description of the epic (or single issue).

## Step 4: Estimate the overall size

Apply the t-shirt sizing rule from `common.md`. State the size, the rationale,
and the resulting structure before continuing.

## Step 5: User stories

Only if the size is **L or larger.** Decompose the work into user stories using
the format and guidelines in `common.md`. If smaller than L, skip to Step 6.

## Step 6: Functional requirements

Produce the numbered FR list per `common.md`.

## Final step: update the issue in Linear

Once the PRD and functional requirements (plus user stories, if **L or larger**)
are written, have the user review and confirm. Only after they confirm, use the
Linear MCP tools to update the **existing** issue, following `common.md` →
"Linear structure reference".

In both cases:

- Update the title to a descriptive but concise name (preserve the existing
  title if it already fits).
- Replace the description with the PRD body and the branch line, if any.
  Keep an existing `[repo=…]` tag only if the user wants it (see `common.md` →
  "Routing and branch lines").
- Preserve the existing labels, priority, assignee and blocking relations
  unless the user requests otherwise. Add `blockedBy` for any new
  prerequisites.
- Preserve any existing comments.

**If L or larger — convert it into a structured epic:**

- Create a sub-issue for each user story, in story order: plain story title,
  story body in the prescribed format, a priority, its `blockedBy`
  dependencies, and the manual label for steps for a person.
- Create the functional-requirements validation sub-issue (format in
  `common.md`), blocked by all the other sub-issues.
- Do the post-creation steps: replace drafting labels with Linear identifiers,
  check the sub-issue order, and verify the blocks relations.

**If smaller than L — keep it as a single self-contained issue:**

- Append the functional requirements and then a consolidated
  `## Acceptance Criteria` checklist as the last section.
- Follow the single-issue formatting rules in `common.md` so omniralph doesn't
  split it into sub-issues.
- Do **not** create any sub-issues.

## Checklist before updating the issue

- [ ] Loaded and summarized the existing issue for the user
- [ ] Asked clarifying questions with lettered options, targeted at actual gaps
- [ ] Incorporated the user's answers into the PRD
- [ ] Estimated the overall t-shirt size and chose the structure accordingly
      (epic only if L or larger)
- [ ] If decomposed: user stories are small, atomic, and specific
- [ ] Acceptance criteria are verifiable (not vague)
- [ ] Functional requirements are numbered and unambiguous
- [ ] Non-goals section defines clear boundaries
- [ ] Success metrics are specific and measurable
- [ ] Quality gates match CI and follow the exact `## Quality Gates` format
- [ ] Every acceptance criterion can be checked by the agent inside the repo;
      steps for a person are separate manual sub-issues
- [ ] If decomposed: every story has a priority and its dependencies as
      blocks relations; no story IDs or metadata in titles or bodies
- [ ] Issue is in the team/project (or has the label) that routes it; no
      `[repo=…]` tag unless the user asked for one
- [ ] No `## Open Questions` section left in any description

Once the issue is updated, offer to delegate it to omniralph, then tell the user
you're done.
