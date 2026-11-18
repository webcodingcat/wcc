---
name: phase
description: Discuss, plan and implement the next open phase from the wcc roadmap test-first and without questions after its start, then stop, so the user starts the following one, best in a fresh session. Use this whenever the user wants to start, continue or resume the implementation of a wcc project, or says "continue with the implementation", "next phase", "next milestone" or "implement the roadmap".
---

# phase: implement one phase

Arguments: $ARGUMENTS

All planning documents live in `.project/`; file names below are relative to
it. Words used here:

- **milestone**: a version of the project (`## Milestone <n>` in
  ROADMAP.md). It ends with all end-to-end tests and a tag.
- **phase**: one slice of a milestone that runs and can be checked; this
  skill builds one. IDs: `01`, `02`, … for planned phases, `F<n>` for
  feedback phases, `B<n>` for backlog phases. Its folder is
  `phases/<id>/`.
- **plan**: one run of the `implementer`, for one component, inside a phase
  (`03-02`). Plans of the same **wave** do not depend on each other and may
  run at the same time.
- **task**: one step inside a plan, one to three per plan.
- **requirement**: one checkable capability in REQUIREMENTS.md (`AUTH-01`),
  with its test level. **Success criteria**: two to five observable results
  of a phase, in ROADMAP.md.

The user starts the phase and may be asked at its start. After that nobody
watches it: the rules of the project CLAUDE.md apply strictly, no
questions, decide and record, block late.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in,
also in the reports; this skill does not set one. A call without text
carries no language: then use the configured one, or the language of the
conversation so far, never the language of the documents or of this skill,
which are English. Everything written to the repository is English.

Read a reference file of this skill only when it applies:
`references/e2e.md` if the project CLAUDE.md has a separate end-to-end
command, `references/backlog.md` for the backlog and `B` phases,
`references/feedback.md` for FEEDBACK.md, `F` phases and findings about the
tests.

## The principle

A phase delivers its success criteria and the requirements it covers, and
proves them with the smallest set of tests that does. Plan the tests before
writing them (`TESTS.md`), write exactly those, and check against that plan.
More tests, more hardening and more backlog than the requirements ask for
make every later phase slower and are not a better result.

## Reading

- STATUS.md: the first line, "Current state" and "Open decisions"; the
  rest only when the status needs it.
- ROADMAP.md: the section of the current milestone. Finished phases there
  are one line each; their details are in ROADMAP-DONE.md, which no skill
  reads.
- REQUIREMENTS.md, SPEC.md, ARCHITECTURE.md, DECISIONS.md: a file of up to
  300 lines in full, a longer one by its headings (list them with Grep):
  the requirements of the phase and every requirement in category `DOC`;
  in SPEC.md "Stack", "Accounts and secrets", "Machine prerequisites",
  "Documentation"; in ARCHITECTURE.md "Overview" and what the phase
  touches; in DECISIONS.md the entries naming a component or requirement of
  the phase and every entry with `Cost of change: high`.
- `research/EXPERIMENTS.md`: the `valid` entries on what the phase uses.

When in doubt whether a section matters, read it. Pass the section names to
the agents; they read those, not the whole files.

Keep what you write short, because every later phase reads it: a
DECISIONS.md entry has one sentence for `What` and one for `Why`;
ARCHITECTURE.md holds what other components build on, not how a component
works inside (that goes into a code comment); STATUS.md lines are one or two
sentences, details go into the phase folder.

## Keeping the main session small

Everything that reaches this session is sent again with every later call.
Build, test and lint output goes to a file; read the summary and the
failures only. Agents return a short result in a fixed form; ask for
nothing more. When an agent is "waiting on its own background work", end
the turn without a tool call. Start your own long runs in the background and
wait for their notice.

## At the start

The user has just typed the command, so asking is cheap now and only now.
Compare the line `wcc version:` in the project CLAUDE.md with `version` in
`.claude-plugin/plugin.json` two folders above this skill folder; if it is
missing or older, recommend `/wcc:upgrade` and ask whether to stop for it or
continue. If FEEDBACK.md exists, record it first (`references/feedback.md`).
Then act on the status:

- `READY` or `IN_PROGRESS`: show the backlog hint (`references/backlog.md`)
  and continue with "Discuss".
- `BLOCKED`: show the reason and the needed decision as one question with
  the options that fit: approve the action (record it in DECISIONS.md with
  `Type: approval`; it covers only that action), decide (record the choice,
  and in ARCHITECTURE.md, SPEC.md or REQUIREMENTS.md if it concerns them),
  done (for what only the user could do: check it is really there), try
  again with a hint (three new attempts), change the requirements or the
  architecture (point to `/wcc:change` and stop), or leave it blocked
  (stop). After the first four, set `IN_PROGRESS`, turn the `[!]` plans
  back to `[ ]` and continue.
- `MANUAL_CHECK`: first fix the findings under "Findings from manual
  testing" in the waiting phase's `PLAN.md`, as plans of that phase on its
  branch. Then run its checks with `Result: open` or `does not fit, see
  plan` as the guided mode of `/wcc:feedback` does (`../feedback/SKILL.md`).
  A check that does not fit and deviates from the requirements is fixed the
  same way and asked again; a wish beyond them goes to `/wcc:change` (stop).
  When none is open, close the waiting phase (step 10) and stop as "After
  the phase" describes. If no phase waits (a milestone ended with open
  checks), only run the checks, then set `DONE`.
- `DONE`: say so; the next milestone is planned with `/wcc:change`, which
  also takes the open decisions (name how many). Stop.
- anything earlier: point to `/wcc:spec` and stop.

### Discuss

Pick the phase (step 1 below). If `phases/<id>/PLAN.md` exists, it was
interrupted: skip this and resume.

Otherwise read the phase's requirements and success criteria and the code
they touch, and work out how you would build it where they leave room: a
behaviour two readings allow, a limit not given, a choice the user will
notice. Present that as one short list of assumptions, "this is how I would
do it", one line each, and let the user correct any of it. Only what you
cannot assume sensibly becomes a question: the open decisions in STATUS.md
that concern the phase, and a real grey area where the options lead to
different results. Ask those in the same round, at most five, each with two
to four options, a recommendation and "I don't know" (take the
recommendation).

Write the confirmed assumptions and the answers to `phases/<id>/CONTEXT.md`,
one line each. An answer that settles an open decision is worked in as
`/wcc:change` does (`../change/SKILL.md`, steps 2 to 5): the documents it
changes, DECISIONS.md with its `Q<n>`, the entry removed. An open decision
the user wants to think about stays open; plans that depend on it are
marked `[!]` with ` - waits for Q<n>`. If there is nothing to assume or ask,
say so in one line.

From here on the phase asks nothing.

## Per phase

1. **Pick** the first unchecked phase of the current milestone in
   ROADMAP.md (never "Later"). Set `IN_PROGRESS`.
2. **Branch** `wcc/phase-<id in lower case>` from the current branch, noted
   as `Base branch` in STATUS.md. The base branch holds only finished
   phases.
3. **Resume.** If `phases/<id>/PLAN.md` exists, an earlier run was
   interrupted: switch to the branch and continue with the first plan not
   marked `[x]`, without planning or writing tests again.
4. **Research.** List what the phase uses for the first time, and where the
   plans depend on exact behaviour of a library, API or tool, unless a
   `valid` entry in `research/EXPERIMENTS.md` covers it. Give the list to
   the `researcher` agent and write its answer, a few lines with sources,
   to `phases/<id>/RESEARCH.md`. Skip this if nothing is new.
5. **Plan** `phases/<id>/PLAN.md`, for you, not for approval:

   ```
   - [ ] 03-01 <title>
     - Wave: 1 | Depends on: - | Review: alone
     - Requirements: AUTH-01, AUTH-02 | Tests: <test files from TESTS.md>
     - Size: ~250 lines in 4 files
     - Adjusts: <existing tests that only use something this plan removes>
     - [ ] <task: what, in which files>
     - [ ] <task>
   ```

   - Every `unit` and `e2e` requirement of the phase is in a plan, and the
     plans together deliver the success criteria.
   - A plan covers one component, one to three tasks, no task touching more
     than five files, about 400 changed lines at most. Split a plan that is
     larger, also one bundling backlog items: an agent sends its whole
     context with every turn, so a large plan costs more than two halves.
     Something done the same way in several components is a shared first
     plan, then one per component.
   - Waves: a plan's wave is one more than the highest wave of the plans it
     depends on; plans of one wave touch different files. A plan that
     depends on another starts only after that one is reviewed.
   - Up to three small plans of one component in consecutive waves,
     together about 300 lines, may share one review: `Review: with 03-01`.
     Plans running at the same time, and plans changing an interface others
     build on, are reviewed alone.
   - If SPEC.md has a "Documentation" section that is not "none", the last
     plan is `Documentation`: it covers the `DOC` requirements and runs at
     the close.
   - A decision with high consequences that REQUIREMENTS.md, SPEC.md,
     ARCHITECTURE.md, DECISIONS.md and CONTEXT.md do not settle is not
     yours: mark the plans that depend on it `[!]` with the question, two to
     four options and your recommendation, and treat them as plans that
     cannot be finished.
6. **Test map.** Write `phases/<id>/TESTS.md` before any test:

   ```
   | Requirement | Behaviour | Level | Test | Plan |
   |-------------|-----------|-------|------|------|
   | AUTH-03 | a reset link older than one hour is refused | unit | internal/auth/reset_test.go: TestReset/AUTH_03_ExpiredLink | 03-02 |
   ```

   Test what the requirements promise, once, at the cheapest level that
   proves it:
   - One row per promise of the requirement: what it states, including the
     error cases it names. Another case only if it happens in normal
     operation (network down, service stopped, file missing, user cancels)
     or its failure destroys data or leaks a secret.
   - The level is the one REQUIREMENTS.md gives: `unit` or `e2e`. A `none`
     or `manual` requirement gets no row.
   - Test a promise where it arises. Code that only passes it on (an API
     client, a handler mapping errors, a UI backend) gets at most one row
     for the whole path.
   - Behaviour many resources or endpoints share (labels, sorting, paging,
     not found, body limits, unknown fields) is one table-driven row over all
     of them, where it is implemented once.
   - No rows for glue code, registration of routes or middleware, data types
     and field tags, configuration without logic, and none that read source
     files or fix how code or tests are structured.
   - Variants of one promise (per image, per input) are one row with a
     table of cases; the full matrix only where the result depends on the
     variant.
   - An `e2e` row whose logic is involved may get a `unit` row for that
     logic as well, so the implementer gets feedback in seconds.
   - The test name carries the requirement ID (`AUTH_03`) and no other, one
     test or subtest per row, in a group named after the component. Test
     files are named after the component and live with the code or in the
     test directory.

   Check the map: every `unit` and `e2e` requirement of the phase, and every
   `DOC` requirement without a test yet, has a row; levels match
   REQUIREMENTS.md; every test name carries its ID. Fix it until it holds.
7. **Tests first.** Give the `test-writer` the rows of `TESTS.md`, the
   section names of their requirements and interfaces, and the existing
   test group to follow where one exists (from the file names, with Glob).
   One writer per component, at most two at the same time, each owning its
   test files (a shared helper or harness belongs to one of them; the
   others report what they need there); a phase with only a few rows gets
   one writer. Writers write exactly the rows: a row they think is missing
   or wrong they report, and you decide. They run only their fast tests and
   compile the end-to-end tests.

   Then check coverage: for every row, find its test name in the test files
   (Grep). A row without its test goes back to the writer; finding no test
   at all is a failure, never a pass. Run the new end-to-end tests once with
   the filter of their IDs, and hand a test that fails for the wrong reason
   back. Add the tests the writers listed as only using something removed
   or renamed under `Adjusts:`. Commit all tests together: a worktree
   contains only what is committed.
8. **Implement** wave by wave. Give each plan except `Documentation` to the
   `implementer` with its plan entry, its rows of `TESTS.md`, the section
   names, `CONTEXT.md`, its `RESEARCH.md` and `research/EXPERIMENTS.md`
   entries, the check commands and UI-SPEC.md if it builds user interface.
   At most two agents run at the same time, counting implementers and
   reviewers; plans of one wave running together each get their own
   worktree. Two plans with end-to-end tests never run at the same time:
   their runs collide in the real environment. Fixes of blockers and of
   failures at the close go to the `implementer` too.
9. **Done means green, then reviewed.** A plan is done when its own tests,
   the other tests of the packages it changed, the linter on those and the
   build pass (the narrower commands of the project CLAUDE.md). Its
   end-to-end tests run once the fast checks are green, by the
   `implementer`, with the filter of its IDs; check in its result that the
   run is green on the commit it names. Tests of open plans may fail.

   Commit, then run the `code-reviewer` once with the plan, its rows of
   `TESTS.md`, the commit and the section names. Fix blockers, write
   decisions to "Open decisions", and take backlog items in as
   `references/backlog.md` says: most reviews add none. Then mark the plan
   and its tasks `[x]`. No second review round.
10. **Close.** Merge the plan worktrees. Run the documentation plan: give
    the `doc-writer` the phase, its requirements, the changed files and
    "Documentation" of SPEC.md, and commit what it wrote; it runs first
    because the verifier and the `DOC` tests check what it writes. Then
    start the unit suite and the linter over the whole project in the
    background, output to files, with the end-to-end selection of
    `references/e2e.md`. While it runs, run the `verifier` with the phase,
    its success criteria and requirements, `TESTS.md`, the changed files and
    the section names (at the last phase of a milestone, all requirements of
    the milestone too); it only reads code.

    Once the suite has ended, fix its failures and the verifier's blockers
    like plans, then rerun as `references/e2e.md` says ("After a fix").
    Write the verifier's decisions to "Open decisions" and take its backlog
    items in like a review's. For each architecture difference: where a
    decision, a plan or an allowed addition explains the code, update
    ARCHITECTURE.md now; where nothing does and the code breaks a documented
    interface or rule, fix it like a blocker; where it is unclear, write an
    open decision. Handle the `manual` requirements (see below). Merge the
    branch into the base branch and delete it. In ROADMAP.md, move the
    phase's full entry unchanged to ROADMAP-DONE.md (create it from
    `../idea/assets/ROADMAP-DONE.md` if missing) and leave one line: `- [x]
    03 <title> (<requirement IDs>) → phases/03/`. Mark its requirements
    done in "Coverage" of REQUIREMENTS.md. Update STATUS.md.
11. **Report**, at most ten lines: what runs now, decisions taken (titles),
    the manual checks waiting, with the hint that `/wcc:feedback` walks the
    user through them, the new open decisions (ID and question) and how many
    are open with `/wcc:change` taking the answers, the backlog hint, and
    the next command.

## Plans that cannot be finished

A plan gets three repair attempts after its first implementation. It stops
at once when a stop condition of the project CLAUDE.md is met. Do not end
the phase then: mark it `[!]` with the reason and what was tried, leave its
branch unmerged, skip the plans that depend on it and finish all others.
Then set `BLOCKED` with the plan, the reason and the exact decision or
action needed, report and stop.

## Shared files and merges

Only this session writes DECISIONS.md, BACKLOG.md, STATUS.md, ROADMAP.md,
REQUIREMENTS.md and the phase folder; implementation agents do not edit the
documents under "Documentation" in SPEC.md either. Agents return decisions
and findings and you write them down. When an agent reports an interface
change, update ARCHITECTURE.md before the next plan starts, searching the
whole file for the name. The git rules for agents are in the project
CLAUDE.md.

Resolve a merge conflict yourself when it is mechanical; otherwise drop the
later plan's branch and redo it on the merged state. A conflict is never a
reason for `BLOCKED`. After a plan's merge, remove its worktree and branch
with `git worktree remove <path>` and `git branch -d <branch>`, never with
`--force` or `-D`; when they refuse, leave them and name them in the
report. Keep the worktree of a plan marked `[!]`. At the start, remove the
same way the worktrees of earlier runs whose branches are merged.

## Manual requirements

The run cannot check `manual` requirements. At the close, write each to
`phases/<id>/CHECKS.md` with a guide the user can follow without knowing the
code, while the code is fresh:

```
## UI-04 <the requirement in one sentence>

- Start: <command from the project CLAUDE.md, URL or window>
- Setup: <data, account, state needed first; what Claude can prepare>
- Steps: 1. <what to click or type> 2. ...
- Expected: <what the user sees when it fits>
- Result: open
```

Check the start command and the steps against what was built. Add one line
per check to "Open manual checks" in STATUS.md: `- UI-04 (03): <requirement>`.

A phase marked `Stops for: manual check` in ROADMAP.md stays unchecked and
unmerged: set `MANUAL_CHECK`, report and stop. Otherwise close it; the
checks wait in the list.

## Open decisions

A finding that would need a change to REQUIREMENTS.md, SPEC.md or
ARCHITECTURE.md, or a wish not taken up yet, is the user's: it goes to "Open
decisions" in STATUS.md, never to the backlog, one question per entry:

```
- Q<n> (<phase and plan, or feedback>): <the question in one sentence>
  - Context: <what was found and where, two or three lines>
  - Options: 1. <option> 2. <option> (two to four)
  - Recommendation: <option>, because <reason>
  - Affects: <requirement IDs, ARCHITECTURE.md sections>
```

Numbers continue from the highest `Q` used, in STATUS.md and DECISIONS.md.
A line `  - In clarification: decisions/<topic>.md` from `/wcc:change` stays
as it is. The entry waits for the next "Discuss" or for `/wcc:change`; until
then the planning blocks every plan that depends on it.

## After the phase

A run implements one phase and stops after its report, also when the next
is a feedback phase it just inserted. The next starts with the user's next
`/wcc:phase`, best in a new session. Set STATUS.md before the report:

- phases of the milestone still open: `IN_PROGRESS`, the next phase under
  "Next".
- the milestone is complete: run all end-to-end tests if they did not run
  at this close (`references/e2e.md`), tag the base branch
  `wcc-milestone-<n>` unless the tag exists, and set `DONE`, or
  `MANUAL_CHECK` if manual checks are open; with open decisions, name
  `/wcc:change` under "Next".
- a plan could not be finished: `BLOCKED`.
- the phase waits for a manual check: `MANUAL_CHECK`.

End the report with the next command: `/wcc:phase` in a new session for
`IN_PROGRESS` and `BLOCKED`, `/wcc:feedback` and then `/wcc:phase` for
`MANUAL_CHECK`, `/wcc:change` for `DONE`. Never start an item of "Later" on
your own.
