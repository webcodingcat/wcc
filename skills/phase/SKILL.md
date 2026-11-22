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
which are English. Results of agents and messages from other sessions
arrive in English, like messages; they do not change the user's language.
Everything written to the repository is English.

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
- `phases/<id>/PLAN.md`: the plan entries and their check marks, when you
  need them; `TESTS.md` only with Grep for one plan's rows.

Do not read REQUIREMENTS.md, SPEC.md, ARCHITECTURE.md, DECISIONS.md,
EXPERIMENTS.md or the code to plan: the `planner` does, and every plan
names the sections its agents read (`Reads:`). Read a section yourself only
to write it (an answer worked in, an interface change, the close).

Keep what you write short, because every later phase reads it: a
DECISIONS.md entry has one sentence for `What` and one for `Why`, and an
entry that revokes or changes an earlier one moves that one to
DECISIONS-DONE.md as the header of DECISIONS.md says (create it from
`../idea/assets/DECISIONS-DONE.md` if missing);
ARCHITECTURE.md holds what other components build on, not how a component
works inside (that goes into a code comment); STATUS.md lines are one or two
sentences, details go into the phase folder.

## Keeping the main session small

Everything that reaches this session is sent again with every later call,
for the hours the phase runs. Build, test and lint output goes to a file;
read the summary and the failures only. Give agents references, not
copies: the plan ID and the phase folder, not the plan text, the rows or
the sections. Agents return a short result in a fixed form; ask for nothing
more. When an agent is "waiting on its own background work", end
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
- `MANUAL_CHECK`: first build the findings under "Findings from manual
  testing" in the waiting phase's `PLAN.md`, as plans of that phase on its
  branch. Then run its checks with `Result: open` or `does not fit, see
  plan` as the guided mode of `/wcc:report` does (`../report/SKILL.md`).
  A check that does not fit and deviates from the requirements is fixed the
  same way and asked again; a wish beyond them is worked in and recorded as
  step 5 of `/wcc:report` does, then built the same way.
  When none is open, close the waiting phase (step 10) and stop as "After
  the phase" describes. If no phase waits (a milestone ended with open
  checks), only run the checks, then set `DONE`.
- `DONE`: say so; the next milestone is planned with `/wcc:change`, which
  also takes the open decisions (name how many). Stop.
- anything earlier: point to `/wcc:spec` and stop.

### Discuss

First check the working tree with `git status`. Changes to the planning
documents in `.project/` left by another skill (FEEDBACK.md aside) belong
to the base branch: commit them there now, before the phase branches
off. Name any other uncommitted change to the user and ask whether to
commit it or leave it; never stash or discard it.

Pick the phase (step 1 below). If `phases/<id>/PLAN.md` already holds
plans, it was interrupted: skip this and resume.

Otherwise start the `planner` with the phase ID and the open decisions in
STATUS.md that concern the phase (step 1 of the agent). If it lists
research, start the `researcher` at once in the background with that list
and the file `phases/<id>/RESEARCH.md` to write. Run the check command of
each tool it lists. Present the planner's
assumptions as one short list, "this is how I would do it", one line each,
and let the user correct any of it while the research runs. Below the
list, name each tool missing with its install command, for the user to
run now; install nothing yourself. End the turn
with the list and ask nothing in it: an open question menu would catch the
user's reply to the list. If the research is done before the user has
replied, say only that it is done; what it changes comes from the planner.
When the research is done and the user has replied, continue the planner with `RESEARCH.md` and the user's corrections;
show the assumptions it changed, one line each. Then ask its questions,
after the list and apart from it: each on its own with AskUserQuestion,
its options, the recommendation and "I don't know" (take the
recommendation); without menus, numbered, saying that each needs an
answer. A reply to the list answers none of them; ask again for those still
open.

Write the confirmed assumptions and the answers to `phases/<id>/CONTEXT.md`,
one line each. An answer that settles an open decision is worked in as
`/wcc:change` does (`../change/SKILL.md`, steps 2 to 5): the documents it
changes, DECISIONS.md with its `Q<n>`, the entry removed. An open decision
the user wants to think about stays open; note it in CONTEXT.md, and the
planner marks the plans that depend on it `[!]` with ` - waits for Q<n>`. If there is nothing to assume or ask,
say so in one line.

From here on the phase asks nothing.

## Per phase

1. **Pick** the first unchecked phase of the current milestone in
   ROADMAP.md (never "Later"). Set `IN_PROGRESS`.
2. **Branch** `wcc/phase-<id in lower case>` from the current branch, noted
   as `Base branch` in STATUS.md. The base branch holds only finished
   phases.
3. **Resume.** If `phases/<id>/PLAN.md` holds plans, an earlier run was
   interrupted: switch to the branch and continue with the first plan not
   marked `[x]`, without planning or writing tests again.
4. **Research** ran during "Discuss". If an answer there raised something
   new to look up, give it to the `researcher` to add to `RESEARCH.md`.
5. **Plan.** Continue the same `planner` (step 2 of the agent) with
   `CONTEXT.md` and `RESEARCH.md`; if it cannot be continued, start it anew
   with both and the phase ID. It writes `phases/<id>/PLAN.md` (plans with
   waves, tasks, sizes and `Reads:`, and the shared "Interfaces") and the
   test map `phases/<id>/TESTS.md`, and returns the waves. Its result is
   what you coordinate with. Record its decisions in DECISIONS.md. Plans it
   marked `[!]` are plans that cannot be finished. For a feedback or
   backlog phase, also give it the path of `references/feedback.md` or
   `references/backlog.md` of this skill; it plans by that file.
6. **Commit** the phase folder on the branch.
7. **Tests first.** Start a `test-writer` per writer the planner named,
   with the phase folder, its test files and the existing test group to
   follow where one exists (from the file names, with Glob); it reads its
   rows in `TESTS.md`, "Interfaces" in `PLAN.md` and the `Reads:` sections
   of their plans. At most two run at the same time, each owning its
   test files (a shared helper or harness belongs to one of them; the
   others report what they need there); parts of one component run one
   after another in the planner's order. Writers write exactly the rows: a row they think is missing
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
   `implementer` with its plan ID, the phase folder and its worktree or
   branch; it reads its entry in `PLAN.md`, its rows of `TESTS.md`, the
   `Reads:` sections, `CONTEXT.md` and `RESEARCH.md` itself, and takes the
   check commands from the project CLAUDE.md.
   At most two agents run at the same time, counting implementers and
   reviewers; plans of one wave running together each get their own
   worktree. Two plans with end-to-end tests never run at the same time:
   their runs collide in the real environment. Fixes of blockers and of
   failures at the close go to the `implementer` too. A plan stopped
   `outside the plan` goes to a new `implementer` with the plan ID, the
   same worktree and the reported failure as what to fix; the plan is done
   once its tests pass after that fix.
9. **Done means green, then reviewed.** A plan is done when its own tests,
   the other tests of the packages it changed, the linter on those and the
   build pass (the narrower commands of the project CLAUDE.md). Its
   end-to-end tests run once the fast checks are green, by the
   `implementer`, with the filter of its IDs; check in its result that the
   run is green on the commit it names. Tests of open plans may fail.

   Commit, then run the `code-reviewer` once with the plan ID (or the
   group), the phase folder and the commits, fixes of failures outside the
   plan included. Fix blockers, write
   decisions to "Open decisions", and take backlog items in as
   `references/backlog.md` says: most reviews add none. Then mark the plan
   and its tasks `[x]`. No second review round.
10. **Close.** Merge the plan worktrees. Run the documentation plan: give
    the `doc-writer` the phase ID, the phase folder and the changed files; it
    runs first because the verifier and the `DOC` tests check what it
    writes. Run once yourself each example it reports as refused by a
    permission check, with the access SPEC.md names, correct the document by
    the real output, and name in the report what is refused again. This is
    no way around the refusal: the check decides anew for your call, in
    the user's view. Commit
    the documents. Then
    start the unit suite and the linter over the whole project in the
    background, output to files, with the end-to-end selection of
    `references/e2e.md`. While it runs, run the `verifier` with the phase
    ID, the phase folder and the changed files (at the last phase of a
    milestone, say so: it checks all requirements of the milestone too); it
    only reads code.

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
    the manual checks waiting, with the hint that `/wcc:report` walks the
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
REQUIREMENTS.md and the phase folder. The `planner` writes `PLAN.md` and
`TESTS.md` first, the `researcher` writes `RESEARCH.md`; after that you
keep them (check marks, `Adjusts:`, plans for findings). Implementation agents do not edit the
documents under "Documentation" in SPEC.md either. Agents return decisions
and findings and you write them down. When an agent reports an interface
change, update ARCHITECTURE.md before the next plan starts, searching the
whole file for the name. The git rules for agents are in the project
CLAUDE.md.

Resolve a merge conflict yourself when it is mechanical; otherwise drop the
later plan's branch and redo it on the merged state. A conflict is never a
reason for `BLOCKED`. After a plan's merge, remove its worktree and branch
with `git worktree remove <path>` and `git branch -d <branch>`, never with
`--force` or `-D`. If the worktree is refused as modified although `git
diff` there shows nothing (a stale index entry after a line-ending
change), stage the files it names with `git add` there and try again;
when they still refuse, leave them and name them in the report. Keep the worktree of a plan marked `[!]`. At the start, remove the
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
`IN_PROGRESS` and `BLOCKED`, `/wcc:report` and then `/wcc:phase` for
`MANUAL_CHECK`, `/wcc:change` for `DONE`. Never start an item of "Later" on
your own.
