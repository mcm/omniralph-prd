# Mode: edit

You are helping revise an existing PRD for a software feature. The existing PRD
may not conform to the standard format — part of your job is to normalize it
during the edit (subject to the user's normalization preference below).

Follow the artifact formats in `common.md`.

---

## Step 1: Load the existing epic

Before doing anything else, use the Linear MCP tools to fetch the parent issue
and all of its sub-issues.

- Identify the parent issue (the epic) and read its full description.
- Enumerate every sub-issue, including the final functional-requirements
  validation sub-issue if one exists.
- Note each sub-issue's identifier, title, description, status, priority,
  labels (especially the manual label), and blocking relationships.
- Note any sub-issues already completed, in progress, or cancelled — these
  constrain what can change.
- Check whether the epic is delegated to omniralph or has an active agent
  session. If it does, omniralph may be working a story right now.
- Look for legacy ralph-tui markup: `US-001:` title prefixes and
  `## Ralph Metadata` sections (Story ID, Ralph Priority, Dependencies).

Then summarize for the user:

- The PRD's current shape (sections present, sections missing, format
  deviations from the standard, including any legacy markup).
- Whether omniralph is running on the epic. If it is, warn that the in-progress
  story is mid-run, and that changes are picked up by replying in the omniralph
  session or by re-delegating.
- The user stories that exist today, in order.
- Whether a functional-requirements sub-issue exists.
- The current state of each sub-issue (completed stories cannot be meaningfully
  edited; in-progress stories should be flagged before changing).

This anchors the conversation — the user may not remember the exact current
state.

## Step 2: Clarifying questions

Ask 3–5 essential clarifying questions about the requested edits. Focus on:

- **Edit Intent:** What is changing and why? Scope change, correction,
  clarification, or pivot?
- **Affected Stories:** Which existing stories are impacted? Any to add, remove,
  or merge?
- **Backward Compatibility:** Are completed/in-progress stories off-limits to
  modification?
- **Format Normalization:** If the existing PRD deviates from the standard
  format, should it be normalized as part of this edit, or left as-is to
  minimize churn?

Use lettered options and adaptive questioning (see `common.md`) — fewer rounds
if the edit is small and well-specified. Example starter set:

```
1. What is the primary nature of this edit?
   A. Scope expansion — adding new functionality
   B. Scope reduction — removing functionality
   C. Correction — fixing errors or unclear requirements
   D. Pivot — fundamentally changing the approach
   E. Format cleanup only — no functional changes
   F. Other: [please specify]

2. How should completed or in-progress sub-issues be handled?
   A. Leave them untouched; only edit pending stories
   B. Edit pending stories and append new ones; touch nothing already started
   C. Reopen and revise specific stories (specify which)
   D. Other: [please specify]

3. Should the PRD be normalized to the standard format?
   A. Yes, fully normalize even if it means rewriting prose
   B. Partial — only normalize the sections being edited
   C. No, preserve the existing structure
```

If the user picks 2C, warn them: reopening a Done story makes omniralph run it
again on the next delegation, on top of code that is already on the epic
branch. Its acceptance criteria should describe the change still needed, not
the whole story again.

For quality gates: if the existing PRD already has a Quality Gates section,
reuse it unless the user wants to change it, but check that it follows the
exact format in `common.md` and still matches CI. If it doesn't have one,
discover the gates from the source you established access to (SKILL.md), CI
config first, and confirm with the user.

## Step 3: Produce the revised PRD

After incorporating the user's answers, generate the revised PRD using the
section format in `common.md`. If the original is missing sections, add them. If
sections deviate from the format, normalize them (subject to the user's
normalization preference from Step 2).

Also add a **Change Log** section (cumulative — preserve any prior entries):

```markdown
### Change Log

- What changed (sections, stories, requirements)
- What is unchanged
- Any sub-issues closed, reopened, added, or whose dependencies shifted
```

Refer to stories in the Change Log by Linear identifier. In a single issue,
place the Change Log before the `## Acceptance Criteria` section, which must
stay last.

## Step 4: Re-estimate the overall size

Apply the t-shirt sizing rule from `common.md`, then compare against the issue's
current structure:

- Already an epic and still **L or larger** → keep the epic structure.
- Currently a single issue but grown to **L or larger** → recommend decomposing
  into an epic, but confirm before restructuring.
- Currently an epic but shrunk below **L** → note this to the user. Collapsing
  an epic back into a single issue is a significant change that must be
  confirmed, and you must never collapse sub-issues that are already completed or
  in progress.

## Step 5: Reconcile user stories (if the issue is/stays an epic)

Reconcile the existing stories with the requested edits. For each story, decide:
**Keep as-is**, **Edit**, **Add**, **Remove** (only if not yet started;
otherwise flag), or **Reorder**. Present the reconciliation explicitly:

```
ENG-12: Keep as-is
ENG-13: Edit — new acceptance criterion for empty state
ENG-14: Remove — superseded by new-2
ENG-15: Keep as-is
new-1:  Add — blocked by ENG-13
new-2:  Add — blocked by ENG-15
```

Guidelines:

- Existing stories are named by their Linear identifier, which never changes —
  omniralph's commits and progress log reference it. Label new stories `new-1`,
  `new-2`, … until they are created.
- Each kept/edited/new story follows the format in `common.md`, with a
  priority, blocks relations, and the manual label where it applies.
- Stories may only be blocked by other stories in the same epic. A new
  dependency on an issue outside the epic goes on the epic, not the story. If
  an existing sub-issue is blocked by an issue outside the epic, flag it and
  propose moving that relation to the epic (`removeBlockedBy` on the story,
  `blockedBy` on the epic).
- **Legacy markup** (if the user chose to normalize): remove `## Ralph Metadata`
  sections and `US-001:` title prefixes from pending stories only. omniralph uses
  a legacy `Ralph Priority` when a sub-issue has no Linear priority, so set the
  Linear priority first. Leave completed and in-progress stories untouched.
- Make sure the functional-requirements validation sub-issue follows
  `common.md`. If it is labelled manual, explain the PR-timing trade-off
  described there and ask whether to keep it that way.

## Step 6: Update functional requirements

Update the numbered FR list. Preserve numbering of unchanged requirements;
append new ones at the end rather than reshuffling, so external references stay
valid.

- "FR-2: When a user clicks X, the system must..." (edited — was about Y)
- "FR-7: [new requirement]"

If a requirement is being removed, mark it deprecated rather than deleting the
number:

- "FR-3: ~~[original text]~~ (removed in this revision)"

## Final step: apply changes in Linear

Present a diff-style summary to the user: PRD sections that changed; stories
kept/edited/added/removed/reordered; FRs added/edited/deprecated. If your size
re-estimate crosses the **L** threshold relative to the current structure, also
summarize the proposed restructuring and get explicit confirmation before
applying it.

Have the user review and confirm before touching Linear. Only after they
confirm, use the Linear MCP tools:

- Update the parent issue's description with the revised PRD body.

**If the issue stays (or becomes) an epic — L or larger:**

- Per story reconciliation: **Keep** → no action; **Edit** → update the
  sub-issue's title/description; **Add** → create a new sub-issue (see
  `common.md` → "Linear structure reference") with its priority, `blockedBy`
  dependencies (sibling sub-issues only) and labels; **Remove** → cancel the sub-issue (do not delete)
  and note it in the Change Log; **Reorder** → update blocks relations
  (`blockedBy` / `removeBlockedBy`) and priorities to reflect the new order.
  Blocks relations carry the order; the sub-issue order in the parent is
  cosmetic, so don't mention it to the user.
- Replace any `new-N` labels in descriptions with the new Linear identifiers.
- Update the functional-requirements validation sub-issue to reflect the revised
  FR list, the stories it covers, and their acceptance criteria. Make sure it is
  blocked by every new story too.
- If a single issue is being promoted to an epic, create the user-story
  sub-issues and the functional-requirements validation sub-issue as in the
  `new`/`triage` final steps.

**If the issue stays (or becomes) a single issue — smaller than L:**

- Keep the functional requirements and a consolidated `## Acceptance Criteria`
  checklist (last section) directly in the issue description, following the
  single-issue formatting rules in `common.md`.
- Do not create new sub-issues. If the user confirmed collapsing an existing
  epic, fold the remaining sub-issues' content into the parent description and
  cancel (do not delete) those sub-issues — but never collapse a sub-issue that
  is already completed or in progress.

Never modify a sub-issue that is already completed, or restructure the issue,
without explicit user confirmation in this session.

## Checklist before applying changes

- [ ] Loaded and summarized the existing epic for the user
- [ ] Asked clarifying questions with lettered options
- [ ] Incorporated the user's answers into the revised PRD
- [ ] Re-estimated the overall t-shirt size and confirmed (or proposed a change
      to) the structure
- [ ] Reconciled every existing story (keep / edit / add / remove / reorder)
- [ ] New or edited stories split by feature, not by phase; each includes its
      own tests, error handling and basic hardening
- [ ] Full scope stated up front; nothing labelled future/stretch/v2 unless the
      user did, and genuinely optional items were put to the user
- [ ] Preserved FR numbers; referred to stories by Linear identifier
- [ ] Acceptance criteria are verifiable (not vague) and can be checked by the
      agent inside the repo
- [ ] Every new or edited story has a priority and its dependencies as blocks
      relations; no story IDs or metadata added to titles or bodies
- [ ] No story is blocked by an issue outside the epic; every outside
      dependency is on the epic itself
- [ ] Quality gates match CI and follow the exact `## Quality Gates` format
- [ ] Warned the user if omniralph is running on the epic or a Done story is
      being reopened
- [ ] Non-goals section still defines clear boundaries
- [ ] Success metrics are specific and measurable
- [ ] Change Log entry written
- [ ] Did not silently modify completed or in-progress sub-issues, or
      restructure the issue without confirmation

Once the changes are applied, if omniralph is running on the epic, tell the
user it picks them up from a reply in its session. Then tell the user you're
done. Don't offer to delegate or re-delegate the epic.
