# Mode: ui-loop

You are creating an epic that improves the UI/UX of a project **in a loop**:
cyralph builds a round of fixes, then a review story critiques the result with
the impeccable design skill and files the next round as new sub-issues. That
continues until the critique clears a quality bar. Write it to Linear.

Invocation: `/linear-prd ui-loop [--lean] [--target <path>] [context]`.

- **Seeded (the default).** You run the baseline critique now, ask the user
  the design decisions it raises, and create the first round of fix stories
  from it. The review story then starts at round 2.
- **Lean (`--lean`).** You skip the critique. The epic holds only the setup
  stories and the review story. Its first pass writes the baseline critique
  and files round 1 as follow-ups. Use it when the user doesn't want to sit
  through a critique now, or wants to make no design calls up front.
- `--target` is the surface to critique (e.g. `src`, `app`, `src/components`).
  If it is missing, infer it from the repo and confirm it.

Follow the artifact formats in `common.md`. This mode always produces an
**epic**, whatever its size.

---

## How the loop works (explain this to the user once)

The review story is an ordinary cyralph story, last in the epic and blocked by
every other sub-issue. When its round finds problems that block the bar, it
reports **blocked** with the problems as `followUps`. cyralph files them as
sub-issues of the epic that block the review story. It builds them, then
re-runs the review story. When a round clears the bar, the story reports
**complete**, and any remaining non-blocking findings join the epic as
ordinary stories. When those are done, cyralph opens the PR.

**Round limit.** cyralph counts each round that reports blocked as a failed
attempt, and a story is set aside after 3 attempts. So a loop stops after 3
rounds unless someone replies `/continue` in the agent session, which gives
the story fresh attempts. Tell the user this when you hand off. (A cyralph
change to give review loops their own round and stall caps is planned; once
it ships, this paragraph no longer applies.)

---

## Step 1: Prerequisites

Check these before anything else, and stop to fix any that fail:

1. **impeccable is installed** in this session (the `impeccable:impeccable`
   skill). If not, tell the user to install the plugin and retry.
2. **impeccable is available to cyralph's sessions.** Story sessions load the
   cyralph host user's `~/.claude` plugins and skills and the repository's
   `.claude/`. Ask the user to confirm the plugin is installed on the cyralph
   host. If it isn't, the review story can't run, so this must be fixed first.
3. **Design context exists.** If the target repo has no `PRODUCT.md`, run
   `/impeccable init`. If it has no `DESIGN.md`, run `/impeccable document`.
   If the user would rather not do this now, make it the first story of the
   epic.
   - **A consumer of a design-system library** that already has its own
     `DESIGN.md`: don't re-document the library's tokens. Write a short
     `DESIGN.md` that names the library's file as the authority and records
     only how this app applies it and where it departs from it. If the epic
     will upgrade the library, describe the target version, not the
     incumbent one.
   - **Both files must be on the default branch before the epic is
     delegated**, because runs branch from it. Commit them on a branch and
     open a PR, or commit to the default branch if the user prefers. Remind
     the user to merge it before handing off.
4. **A screenshot loop exists.** The review story and every UI story need to
   render the UI and screenshot it unattended: a showcase, Storybook, or the
   app's dev server, plus a committed script that captures named screens at
   given viewports and themes. Look for one. If there is none, the epic's
   first story builds it.
   - Ask how the cyralph host runs a browser: a Playwright MCP, a local
     Playwright/Chromium, or a container such as
     `mcr.microsoft.com/playwright:<version>`. Ask this with your Step 1
     questions, not at the end.
   - **An app behind sign-in or backed by a database** needs two setup
     stories, not one:
     1. A fixture seed with stable IDs that covers every state the screens
        show, plus a dev-only sign-in. Gate it on an env var **and** a
        non-production check, with a test that proves it is off in
        production.
     2. The screenshot command that uses it, with a committed route list,
        a scroll-width check at the narrowest viewport, and a `--strict`
        flag for the review story.
   - Record the answer, and any quirks (remote Docker daemon → `docker cp`
     instead of bind mounts), in the epic's Technical Considerations.
   - If the target's `LINEAR.md` forbids browser sessions in acceptance
     criteria, the script must be a plain repo command (e.g.
     `pnpm verify:screens`) that runs headless and writes files. That still
     counts as an in-repo check.

## Step 2: Clarifying questions

Ask these with lettered options (see `common.md`). Skip any the context
answers:

1. **The bar.** Ask for the critique score that ends the loop. Default: 32/40
   with zero P0 and zero P1. In seeded mode, propose baseline + 8 to 10,
   capped at 34.
2. **Viewports and themes** to verify. Default: 1440px and 390px, and every
   theme the project ships.
3. **Blocking severity.** Ask what a failing round files. Default: a round
   fails on any P0 or P1 (or the score below the bar). A failing round files
   follow-ups for P0, P1 and P2; P3 is listed in its report and not filed.
   A passing round reports complete and files nothing more.
   - Some users want everything fixed while the loop is still running:
     a failing round also files P2s, or P3s. If so, bundle the P3s into one
     polish follow-up per round, so they cost one story rather than several.
   - cyralph makes **every** follow-up of a failing round block the review
     story. So whatever a failing round files is built before the next round,
     and a completing round should file nothing that isn't meant to join the
     epic as more work.
4. **Guardrails.** Ask whether the epic should first add automated checks that
   stop regressions between rounds. Examples: lint rules banning raw colours,
   arbitrary sizes and durations outside the token layer, and a WCAG contrast
   check over token pairs. Recommend them for a design system or component
   library. For a small app, they are optional.
5. **Scope fence.** Ask what the loop must not turn into. Default: no new
   features, pages or component families; no palette or font changes; no
   dependency upgrades. Findings that need these are P3 notes, not follow-ups.
   Without a fence, each round's fresh eyes find new feature work, and the
   loop doesn't converge.

Discover the quality gates as `common.md` describes.

## Step 3 (seeded only): Baseline critique and decisions

If Step 1 found no screenshot loop, a seeded baseline can't see the running
app. Offer three options:

- **Lean.** Switch to lean mode, so round 1 makes the full visual baseline
  once the setup stories land.
- **Code-only seed.** Critique the code and run `impeccable detect` now, and
  seed the fix stories from that. Round 1 still writes the first visual
  baseline.
- **Run it now.** The user runs the app locally for a full critique.

1. Run `/impeccable critique <target>`. Use its normal dual assessment, with
   screenshots at the agreed viewports and themes.
2. Make sure the report is saved at
   `.impeccable/critique/<timestamp>__<slug>.md`, then ask the user to commit
   it, or commit it yourself with their OK. The loop finds the previous review
   through git history, so the baseline **must be committed** before the epic
   runs.
3. Note the score, and the P0/P1 counts.
4. The critique raises design calls that only the user can make: direction,
   default theme, what the signature element means, which convention wins
   when two compete. Ask them as lettered questions, one round, and record
   each answer in the epic's Design Considerations. The fix stories and every
   later round are bound by them.

## Step 4: Write the epic

Write the PRD with `common.md`'s sections, plus the following:

- **Introduction.** Describe the product in a sentence. In seeded mode, give
  the baseline score, the critique path and the 4 to 6 headline findings. In
  lean mode, say the first review round writes the baseline.
- **Goals.** Clear the bar (score, P0, P1), plus any measurable goals from
  the decisions, e.g. "every text pair meets WCAG AA in both themes".
- **Quality Gates.** The discovered commands, in exact format, and the guard
  commands once their stories exist ("once US-00x has landed" is fine in
  prose). For UI stories, add a prose bullet like this: "Run `<screenshot
  command>` and inspect screenshots of the changed screens at <viewports> in
  <themes> before reporting complete." Write it as prose, not a leading
  backtick, so it isn't run as a gate.
- **Non-Goals.** The scope fence from Step 2.
- **Design Considerations.** Name `DESIGN.md` and `PRODUCT.md` as the design
  authority. List the decisions from Step 3, and say these rules bind every
  round.
- **Technical Considerations.** Where the tokens and styles live, how the
  screenshot loop runs and its quirks, and how consumers pick up the change.
- **Size.** XL is typical. The epic shape is fixed regardless of size.
- Routing and branch lines as in `common.md`.

## Step 5: Stories

In story order:

1. **Setup** (only what Step 1 found missing): commit the design context,
   build the screenshot loop.
2. **Guardrails** (if chosen): the lint guard rules, the contrast check. Both
   go first, so every later story is held to them.
3. **Seeded only, round-1 fixes.** Group the baseline's P0, P1 and in-scope
   P2 findings into stories of one session each, by layer: tokens first, then
   components, then pages/screens. Then group consumers and docs. Each
   criterion points to a finding and is machine-checkable: a lint rule, the
   contrast script, a test, a grep, or a named screenshot inspected.
4. **The review story**, last, blocked by every other sub-issue. Title:
   `Validate the UI/UX bar and functional requirements`. Keep the leading
   "Validate" so cyralph's triage recognises it as the validation story.
   Body: the template below.

Then write the functional requirements (`common.md`). In a UI loop they are
the epic's lasting invariants: what must stay true after any round. Examples:
the contrast check passes in every theme; no raw colours in components; no
horizontal scroll at the narrowest viewport; `impeccable detect` exits 0.
Don't use one-off fixes as FRs.

### Review story template

Fill in the `<…>` placeholders. Keep the rest verbatim: the anchoring rules
are what make the score track progress rather than reviewer variance.

```markdown
As the project owner, I want each round of UI/UX work reviewed against the last one, so that the loop files the next round of fixes and ends once the bar is cleared.

This story reviews; it doesn't fix. Everything it finds is reported as follow-ups.

## Each round

1. **Gates.** Run every quality gate, the screenshot command and `impeccable detect` on <target> (and on each consumer listed in the epic). Record the output.
2. **Requirements.** Verify FR-1 … FR-n and every completed story's acceptance criteria, with evidence (command output or screenshot path).
3. **Critique.** Run `/impeccable critique <target>` at <viewports> in <themes>, anchored to the previous review as below. <Lean mode only: if `.impeccable/critique/` holds no `*__<slug>.md` yet, this is the baseline round: run a normal, unanchored critique.>
4. **Keep the report.** Leave it in the worktree under `.impeccable/critique/`. cyralph commits it with the round (as `wip(…)` when the round reports blocked), and the next round anchors to it.
5. **Post one comment** on this issue: `Round N: PASS|FAIL`, the score and its trend across rounds, P0/P1/P2 counts, the gates table, the FR table, and the follow-ups filed.

## Anchoring the critique

This overrides the critique skill's default, which shows Assessment A no earlier runs.

1. **Find the previous review.** It is the newest `.impeccable/critique/*__<slug>.md`. Find the commit that added it with `git log -1 --format=%H -- <file>`. The code changed since then is `git diff --stat <commit> -- <target> <other paths>`.
2. **Give Assessment A the previous review and that diff.** Assessment B stays isolated. A must:
   * **Re-check every previous Priority Issue and heuristic key issue.** Mark each *fixed*, *still open* or *regressed*, with a file:line or screenshot as evidence.
   * **Start each heuristic at its previous score.** Change a score only for a stated reason: a previous issue fixed (up), or a regression or new problem in code changed since the previous review (down).
   * **Report problems in unchanged code as *newly noticed*.** A newly noticed P0 or P1 can lower its heuristic. A newly noticed P2 or P3 is listed but doesn't change the score.
3. **Show the comparison.** The report header names the previous review. A *Changes since the previous review* table lists each earlier issue's status and each score change with its reason.

Detector findings checked and judged false positives go in `.impeccable/critique/ignore.md` with the reason, so later rounds don't re-litigate them.

## What to report

* **The bar is cleared** (score ≥ <N>/40, zero <blocking severities>, every gate and FR passes): report **complete** with no follow-ups. List any remaining lower-severity notes in the comment only; the loop is done.
* **Otherwise** report **blocked**, with follow-ups for every <filed severities> finding and every failing FR or gate.
  * Group findings into at most 8 follow-ups, each one session of work, layer by layer. Each states its finding, the evidence and checkable acceptance criteria.
  * Reuse an open sub-issue that already covers a finding rather than filing a duplicate.
  * Anything outside the epic's Non-Goals is a P3 note, never a follow-up.
  * A follow-up for a person (e.g. a commit in another repository) is marked manual. File it only if this story can't pass without it.

## Acceptance Criteria

- [ ] The round's critique, anchored as above (or the baseline in round one), is saved under `.impeccable/critique/`
- [ ] Score ≥ <N>/40 with zero <blocking severities> findings
- [ ] FR-1: …
- [ ] All acceptance criteria of the epic's completed stories still hold
- [ ] `impeccable detect` exits 0 on <target> (and listed consumers)
- [ ] All quality gates pass
```

## Final step: save to Linear

Show the user a summary table before writing: the size, the stories with
priority and blockers, the bar, and the mode. Then follow `mode-new.md` →
"Final step" for an epic. That covers the epic issue, the sub-issues in story
order, the review story blocked by all of them, and the post-creation steps.

Hand off as in `common.md`. Tell the user two things:

- It runs at most 3 rounds before it needs a `/continue` (see "Round limit").
- To review the loop's progress, check the review story's comments.

## Checklist before creating the issues

- [ ] impeccable confirmed installed here and on the cyralph host
- [ ] `PRODUCT.md`/`DESIGN.md` committed, or a setup story creates them
- [ ] A screenshot command exists, or a setup story builds it; host browser method recorded
- [ ] Seeded: baseline critique committed, decisions recorded in Design Considerations
- [ ] The bar, the blocking severity and the scope fence are in the epic
- [ ] Review story is last, blocked by every other sub-issue, and its placeholders are filled in
- [ ] FRs are lasting invariants, not one-off fixes
- [ ] Everything in `mode-new.md`'s checklist for an epic
