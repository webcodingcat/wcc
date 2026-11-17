---
name: milestone
description: Plan and implement the next open milestone from the wcc roadmap test-first and without questions, then stop, so the user starts the following one, best in a fresh session. Use this whenever the user wants to start, continue or resume the implementation of a wcc project, or says "continue with the implementation", "next milestone" or "implement the roadmap".
argument-hint: "[--parallel <n>]"
---

# milestone: implement one milestone unattended

Arguments: $ARGUMENTS

All planning documents live in `docs/wcc/`; file names below are relative to it.

Once the run is going, the user is not watching. A question would stall it for
hours, so the rules in the project CLAUDE.md apply strictly: no questions,
decide and record, and a task that cannot be finished blocks late, not at once.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in,
also in the reports; this skill does not set one. A call without text
carries no language: then use the configured one, or the language of the
conversation so far, never the language of the documents or of this skill,
which are English. Everything written to the repository is English: plans,
planning documents, code, comments and commit messages.

## At the start

The user has just typed the command, so this is the one moment where asking is
cheap. Read STATUS.md and ROADMAP.md; read SPEC.md, ARCHITECTURE.md and
DECISIONS.md as "Reading the planning documents" describes. Do not rely on
earlier conversation. Compare the line `wcc version:` in the
project CLAUDE.md with `version` in `.claude-plugin/plugin.json` two folders
above this skill folder. If it is missing or older, the project files may
lack keys this skill looks for: say so, recommend `/wcc:upgrade`, and ask
whether to stop for it or continue anyway. If FEEDBACK.md exists, record its
findings first (see "Feedback"). Then act on the status:

- `READY` or `IN_PROGRESS`: if "Open decisions" in STATUS.md has entries,
  list them, one line each with ID and question, marking those `In
  clarification`, and ask once: decide them now or continue. To decide, point to `/wcc:change`, which takes the
  answers, and stop. Otherwise, and when nothing is open, continue with the
  preflight, without further questions. Tasks that depend on an open
  decision block in the planning (step 5), so continuing is safe.
- `BLOCKED`: show the reason and the needed decision as one question with the
  options that fit:
  - approve the action: record it in DECISIONS.md with `Type: approval`. The
    approval covers only the action named.
  - decide: for an open decision, offer the options with your recommendation;
    record the user's choice in DECISIONS.md and, if it concerns them, in
    ARCHITECTURE.md or SPEC.md.
  - done, for something only the user could do (a credential, an account, an
    install): check that it is really there now.
  - try again with a hint from the user: the task gets three new attempts.
  - SPEC.md or ARCHITECTURE.md has to change: point to `/wcc:change` and stop.
  - leave it blocked: stop.

  After the first four, set `IN_PROGRESS`, turn the `[!]` tasks in the plan
  back to `[ ]` and continue.
- `MANUAL_CHECK`: ask about every check of the waiting milestone under
  "Open manual checks" in STATUS.md, "fits" or "does not fit" (offered in
  the user's language). A check marked ` - does not fit, see plan` was
  already reported with `/wcc:feedback`: take the findings from the plan
  instead of asking.
  - fits: remove it from the list.
  - does not fit: let the user describe what is wrong. If the code deviates
    from SPEC.md, fix it now like a task of the waiting milestone, on its
    branch, and ask again. If the user wants something other than what
    SPEC.md says, point to `/wcc:change` and stop.
  - When none of its checks is open any more: close the milestone that was
    waiting (step 9), then set `IN_PROGRESS`, or `DONE` if the stage is
    complete. That close is the milestone of this run: report and stop as
    "After the milestone" describes.
- `DONE`: say so. The next stage is planned with `/wcc:change`, which also
  takes up the open decisions, if any; name how many. Stop.
- anything earlier: point to `/wcc:spec` and stop.

## Options

The section "Options" in STATUS.md holds the user's settings for the run;
`/wcc:options` sets them. Read them at the start of the run. A missing
line means its default; a missing section means an older project, so look
for `Parallel agents` under "Current state" as well. A line `Stop after each
milestone` from a project before version 0.25.0 has no effect: every run
stops after one milestone.

- `Parallel agents` (1 to 5, default 1): see "Parallelism".
- `Backlog milestone after` (0 to 10, default 3): see "Backlog milestones".
- `Full e2e run` (`every milestone` or `backlog and stage end`, default
  `every milestone`): see "End-to-end tests".

Arguments of this call apply to this run. Only `--parallel` is also written
back, because the limit is the user's choice for all following runs.

## Parallelism

How many implementation subagents may run at the same time is the user's
choice: parallel agents finish sooner, but they use up the usage limits of the
account just as much faster, and a run that hits the limit stands still.

- `--parallel <n>` was given, with `n` a whole number from 1 to 5: write it to
  STATUS.md as `- Parallel agents: <n>` under "Options", creating the
  section as `/wcc:options` describes if it is missing. For any other value,
  say which values are allowed and stop; the user is still there to correct it.
- Otherwise use the value of `Parallel agents` in STATUS.md, and 1 if the
  line is missing.

The value lives in STATUS.md and not in this conversation, so it survives
interruptions and holds for every following run until the user gives
another one. With 1, nothing runs in parallel and no worktrees are
needed: every task is implemented one after another on the milestone branch.

## Reading the planning documents

SPEC.md, ARCHITECTURE.md, DECISIONS.md and EXPERIMENTS.md grow with every
milestone. Read in full by every milestone and every reviewer, they fill the
context before the work begins, and each milestone takes longer than the one
before. So read a file of up to 300 lines in full, and a longer one by its
headings: list its heading lines with Grep, then read the sections this
milestone needs.

- SPEC.md: the requirements named under `Covers:` in the ROADMAP entry (for
  a backlog or feedback milestone, the ones its items or findings name), and
  the sections that hold for every milestone: "Stack", "Accounts and
  secrets", "Machine prerequisites", "Documentation", "Out of scope".
- ARCHITECTURE.md: "Overview", and the sections on every component,
  interface, fixed rule and flow the milestone touches.
- DECISIONS.md: the entries whose title names a component, requirement or
  term of the milestone, and every entry with `Cost of change: high`
  (search for that line and read its entry).
- EXPERIMENTS.md: the `valid` entries on what the milestone uses.

Where the headings do not tell, search the file for an identifier (a
requirement ID, a type, a file path). When in doubt whether a section
matters, read it: overlooking a decision costs more than one more section.
Select afresh in every run; a selection from earlier conversation does not
carry over.

Pass the selection on. The `test-writer`, `code-reviewer` and
`milestone-verifier` get the names of the sections that concern their task
and read those, not the whole files; they look up further sections by
heading when the change reaches further.

Keep what you add short, because every later milestone reads it. `What` and
`Why` of a DECISIONS.md entry are one sentence each. ARCHITECTURE.md holds
what other components build on: components, their interfaces with their
meaning, data schema, formats, flows. How a component does its work inside
(timeouts, step orders, retries, the reason for a workaround) belongs in a
comment at that place in the code, where whoever changes it finds it, and
in the plan.

## Keeping the main session small

Everything that reaches this session stays in its context until the run
ends and is sent again with every later call. Logs and long reports there
cost the user's usage limits many times over and push out what matters.

- Run every build, test and lint command with its output in a file, here and
  in every subagent, and read only the summary and the failures. Never read a
  whole log of a passing run.
- Ask every subagent you start for a short result in a fixed form, nothing
  else: no logs, no code, no repetition of its task, no account of how it
  worked. For an implementation task:

  ```
  Status: green | not green after <n> attempts | stopped: <reason>
  Commit: <hash>
  Files: <changed files, one line>
  Decisions: <one line each: what, why, cost of change>
  Interface changes: <one line each, additions included; or none>
  Findings: <one line each, for BACKLOG.md or "Open decisions"; or none>
  Failures: <only if not green: failing tests with the decisive message line>
  ```

  The agents in `agents/` have their own result forms; pass them nothing
  that asks for more.

## Preflight

The folder is a git repository with a clean working tree, and the build and
test commands in the project CLAUDE.md are filled in and succeed. If not, do
what `/wcc:prepare` describes. If a tool is missing on the machine, or
SPEC.md names elevated rights and this session does not have them, set
`BLOCKED` naming what is missing and stop.

## Per milestone

1. **Pick.** Pick the first unchecked entry of the current stage in ROADMAP.md. The
   current stage is the first section with unchecked entries, never "Target
   vision (rough)". Set STATUS.md to `IN_PROGRESS`. An entry `B<n>` is a backlog
   milestone, an entry `F<n>` a feedback milestone, see below.
2. **Branch.** Work on `wcc/m<n>`, created from the current branch; note that
   branch as `Base branch` in STATUS.md. The base branch then only ever
   holds finished milestones, also when a run ends blocked.
3. **Resume.** If `plans/M<n>.md` already exists, an earlier run was
   interrupted. Do not plan again and do not write tests again: switch to the
   branch and continue with the first task not marked `[x]`.
4. **Research.** Read EXPERIMENTS.md if it exists, as "Reading the planning
   documents" describes: the feasibility check may already have tried what
   this milestone needs. Use the entries marked `valid` and do not find out
   again what they show. Then list every
   library, API or tool this milestone uses for the first time, and every point where the plan depends on exact behaviour of
   one. Give the list to the `researcher` agent. Plan with what the
   documentation says today, not with what you remember. Skip this only if
   the milestone uses nothing new.
5. **Plan.** Write `plans/M<n>.md` in English. First the research results in
   a few lines with their sources, then small tasks as a checklist: `- [ ] T1
   <title>`, each with the acceptance criteria it covers, the files it touches
   and the tasks it depends on. Every `auto` and `browser` criterion of the
   milestone must appear in a task. Test files a task touches go where the
   project keeps its tests, never into `docs/` or another folder for
   documentation, assets or configuration, even when the task writes a
   document there. This plan is for you. Do not present it for approval.

   Small tasks in the same component that run one after another may share
   one review: mark the later ones `Review: with T<n>`, naming the first of
   the group. At most three tasks, together a change of about 300 lines.
   For a task of a few lines, starting the reviewer and reading its
   sections costs more than it finds; over a large change it overlooks
   more. Tasks in parallel worktrees, and tasks that change an interface
   other tasks build on, are reviewed on their own.

   If SPEC.md has a section "Documentation" that is not "none", the last
   task is always `Documentation`: it depends on every other task, covers
   the criteria `D<n>` and names the documents this milestone affects. It
   runs at the close (step 9), after the merge, because documentation
   describes the integrated result and parallel tasks editing the same
   document would collide. Projects without that section have no such task.

   While planning, look for decisions with high consequences that this
   milestone needs and that SPEC.md, ARCHITECTURE.md and DECISIONS.md do not
   settle. Planning is the place to find them: nothing is built yet. Do not
   take such a decision yourself. Mark the tasks that depend on it `[!]`
   with the question, two to four options and your recommendation, and
   treat them as tasks that cannot be finished (see below). The same holds
   for a task that depends on an entry under "Open decisions" in STATUS.md:
   mark it `[!]` with ` - waits for Q<n>`.
6. **Tests first.** Give the `test-writer` agent the acceptance criteria of
   this milestone, the criteria `D<n>` from "Documentation" that have no
   test yet, and the names of the SPEC.md and ARCHITECTURE.md sections with
   those requirements and their interfaces. It
   writes the tests without seeing any implementation. The tests must fail
   before implementation starts. Commit them right away: a worktree contains
   only what is committed, so uncommitted tests would not reach the subagents.

   With `Parallel agents` above 1, split the criteria by component into
   groups whose tests go into different test files, and give each group to
   a `test-writer` of its own, at most as many at once as `Parallel agents`
   allows. Writing the tests of a large milestone took one writer over
   twenty minutes. Split only where each group has several criteria; for a
   handful, one writer is faster than several that each read their sections.
   All writers work in this checkout, without worktrees, and do not commit,
   so they must not touch the same file:
   - Name for each group its test files. A shared file (a helper, the
     harness of the end-to-end tests, a fixture) belongs to one group only;
     the others report what they need there instead of editing it, and the
     owner's writer or you add it afterwards.
   - The criteria `D<n>` go to one group.
   - Writers run only the fast tests of their group. Two runs of end-to-end
     tests at the same time can collide in the real environment, so they
     only compile those. Once all writers are done, run the new end-to-end
     tests yourself, once, with the filter of their criterion IDs, and hand
     a test that fails for the wrong reason (it does not compile, the setup
     breaks) back to a `test-writer` for its group.

   Then commit all tests together.
7. **Implement.** Give each task except `Documentation` to a subagent with only its task, the
   relevant interfaces, the research results and EXPERIMENTS.md entries it
   needs, the tests it has to
   satisfy, the commands of its checks (step 8), and UI-SPEC.md if it
   builds user interface. Ask for its result in the form under "Keeping the
   main session small". Its interface changes list every change to what
   other components use, an added field, parameter, return value or
   endpoint included: such additions are allowed, but ARCHITECTURE.md
   learns of them only from that list. Run tasks in
   parallel, each in its own worktree, only if they do not depend on each
   other and touch different files, and never more at once than the value
   of `Parallel agents` in STATUS.md (see "Parallelism"). Everything else
   runs one after another. The limit counts every subagent that is running,
   so the `code-reviewer` of a finished task takes one of the places.
8. **Done means green, then reviewed.** A task is done when its own tests,
   the other tests of the packages or modules it changed, the linter on
   those and the build pass. The whole unit suite and the linter over the
   whole project run at the close (step 9): run after every task and every
   repair attempt, they cost more time than anything else in the
   implementation and rarely find what the narrower run misses. Use the
   narrower form of the commands in the project CLAUDE.md (a package path, a
   directory, the test filter of step 6); if a tool takes no such argument,
   run its full command. Tests of tasks that are still open may fail.

   Commit, then run the `code-reviewer` agent once with the task, its
   criteria, the commit and the names of the SPEC.md and ARCHITECTURE.md
   sections the task concerns. Tasks the plan groups for one review
   (step 5) are reviewed together once the last of them is green; a task
   outside the group that depends on one of them waits for that review.
   Fix blockers, write the
   decisions to "Open decisions" (see there), send everything else to
   BACKLOG.md, and only then mark the task `[x]` in the plan. No second
   review round.
9. **Close.** Merge the task worktrees. Then run the documentation task:
   give the `doc-writer` agent the milestone, its criteria, the changed
   files and the section "Documentation" of SPEC.md, commit what it wrote
   and treat it like any other task (step 8). What it could not document
   goes to BACKLOG.md; behaviour it found to contradict SPEC.md is a bug to
   fix like a task. Then start the full test suite and the linter over the
   whole project in the background, with their output in files; now every
   test and every lint check must pass. Which end-to-end tests
   belong to it depends on the option `Full e2e run` (see "End-to-end
   tests"). While it runs, run the `milestone-verifier` agent once, with
   the milestone, its criteria, the changed files and the names of the
   SPEC.md and ARCHITECTURE.md sections they concern: it checks that
   nothing is a stub, everything is wired and the documentation matches
   what was built. It only reads code, so it does not need to wait for the
   tests, which can take an hour. At the last entry of a stage, also give
   it all requirements of the stage, so it checks the documentation as a
   whole before the stage is tagged. Fix its blockers and the failures of
   the suite and the linter like tasks, once the suite has ended, so the fixes do not
   change the code under the running tests. Then run the tests that failed
   and the tests the fixes touch, and the whole selection once more only if
   the suite had failures. Write the verifier's decisions to "Open
   decisions", send the rest to BACKLOG.md. For each of its architecture
   differences: where the code is right because of a recorded decision, a
   task of the plan or an allowed addition, update ARCHITECTURE.md now
   (catching up with what was built is not a change of the architecture,
   also in a backlog milestone); where the code breaks a documented
   interface or rule with nothing to explain it, fix it like a blocker;
   where it is unclear which side is right, write an entry to "Open
   decisions". Handle the `manual` criteria as described
   below. Then merge `wcc/m<n>` into the base branch, delete it, check the
   milestone off in ROADMAP.md and update STATUS.md.
10. **Report** to the user, at most ten lines: what runs now, decisions taken
    (titles from DECISIONS.md), the `manual` criteria the user has to
    look at, with the hint that `/wcc:feedback` shows how to test them and
    takes their results and anything else the user notices, and the
    decisions added to "Open decisions" by this milestone (ID and question),
    with how many are open in all and that `/wcc:change` takes the answers,
    and the next command (see "After the milestone").

## End-to-end tests

End-to-end tests are the tests the project CLAUDE.md runs with a command of
their own under "Build and test commands", because they need the real
environment or a browser and take long. If there is no such command, every
test belongs to the full test suite and this section does not apply.

- `Full e2e run: every milestone` (the default): the close of every
  milestone runs all of them.
- `Full e2e run: backlog and stage end`: the close of an ordinary milestone
  runs only the end-to-end tests whose name carries a criterion ID this
  milestone covers or re-checks (see the plan), and those in test files this
  milestone changed. The close of a backlog milestone and of the last entry
  of a stage runs all of them, so the base branch is fully checked before a
  stage is tagged.

How to select them at the close of an ordinary milestone: list the names of
the end-to-end tests and their subtests, pick the ones above, and turn them
into the filter of the test framework (`-run` in Go, `-k` in pytest, `--grep`
in Playwright, `-t` in Jest), appended to the end-to-end command of the
project CLAUDE.md. Match an ID exactly: `R1.1` must not select `R1.10`. Where
a test holds the criteria of several milestones as subtests, select the
subtests, not the whole test. If nothing is selected, no end-to-end test
runs. Never run the end-to-end command without the filter at such a close,
not even to be safe: that is the full run the user chose not to wait for.
The one exception is a change to the shared harness that every end-to-end
test depends on (its setup, its teardown); then run all and say why. Note in
STATUS.md under "Last done" the exact command that ran, or that none ran.

A full run that fails at a backlog milestone or a stage end found a
regression of an earlier milestone: fix it like a task of the milestone that
is closing.

After a fix, first run only the tests that failed, then the whole selection
once more. Output goes to a file as "Keeping the main session small"
describes.

## Tasks that cannot be finished

A task gets three repair attempts after its first implementation. If its
tests, the linter or the build still fail after the third, stop working on it.
The same applies at once when one of the stop conditions in the project
CLAUDE.md is met: a missing credential, missing elevated rights, a package
that cannot be installed, a destructive action, an open decision with high
consequences, a change to SPEC.md or ARCHITECTURE.md beyond what the project
CLAUDE.md allows.

Do not stop the run at that point. Block late:

1. Mark the task `[!]` in the plan with the reason and what was tried. Leave
   its branch unmerged.
2. Skip the tasks that depend on it. Finish all others.
3. Then set STATUS.md to `BLOCKED` with the task, the reason and the exact
   decision or action needed from the user, report and stop. The milestone
   stays unchecked and its branch unmerged.

This way one stuck task costs the user one decision, not the whole run.

## Shared files and merges

Subagents never write DECISIONS.md, BACKLOG.md, STATUS.md, ROADMAP.md or the
plan. Implementation subagents do not edit the documents listed under
"Documentation" in SPEC.md either; the documentation task does that after the
merge. Two worktrees editing the same file would collide on merge. They return
their decisions and findings in their result, and the main session writes them down. If a
subagent extended an interface, update ARCHITECTURE.md before the next task
starts, with the change to the interface and its meaning, not how it is
implemented (see "Reading the planning documents"). Search the whole file
for the name of what changed: the same interface or rule is often named in
more than one section, and the one left behind contradicts the code. The
`milestone-verifier` catches what slips through at the close. The git rules for subagents are in the project CLAUDE.md.

If a merge conflicts anyway, resolve it in the main session when it is
mechanical. Otherwise drop the later task's branch and redo that task on top of
the merged state. A merge conflict is never a reason for `BLOCKED`.

## Manual criteria

`manual` criteria cannot be checked by the run. At the close of a milestone,
add them to "Open manual checks" in STATUS.md, each with a test guide the
user can follow without knowing the code:

```
- R2.3 (M2): <the criterion in one sentence>
  - Start: <command from the project CLAUDE.md, URL or window to open>
  - Setup: <test data, account, state needed first; omit if none>
  - Steps: 1. <what to click or type> 2. ...
  - Expected: <what the user sees when it fits, from SPEC.md and UI-SPEC.md>
```

Write the guide now, while the code of the milestone is fresh: the names on
the screen, the route, the key to press. Check that the start command and
the steps match what was built. `/wcc:feedback` without arguments shows
these guides to the user.

- The milestone is marked `Stops for: manual check` in ROADMAP.md: leave it
  unchecked and its branch unmerged, set STATUS.md to `MANUAL_CHECK`, report
  and stop. The next call of this skill closes it with the user, as described
  under "At the start".
- Otherwise: close the milestone. The criteria wait in the list, and the
  report names them.

## Open decisions

Some findings are not the run's to settle: a review, the verifier, a backlog
milestone or the user's feedback finds something that would need a change to
SPEC.md or ARCHITECTURE.md, or a wish the user has not taken up yet. They go
to "Open decisions" in STATUS.md, not to BACKLOG.md. A backlog milestone
works off its items unattended and cannot settle such a point, and in the
backlog nobody sees it. STATUS.md is where the user looks for what waits for
them.

Write each entry so the user can decide it without reading the code, one
question per entry. Split a finding that mixes several questions, and leave
the parts that need no decision in BACKLOG.md.

```
- Q<n> (<milestone and task, or feedback>): <the question in one sentence>
  - Context: <what was found and where, in two or three lines>
  - Options: 1. <option> 2. <option> (two to four)
  - Recommendation: <option>, because <reason>
  - Affects: <requirement IDs, ARCHITECTURE.md sections>
```

`/wcc:change` may add a line `  - In clarification: decisions/<topic>.md`
while the user discusses the entry in a separate session. The entry is
still open; leave the line as it is.

Numbers continue from the highest `Q` ever used; search DECISIONS.md for
`Q<n>` too, because entries are removed once decided. Writing down options and a
recommendation is not deciding: the entry waits until the user answers with
`/wcc:change`, which updates the documents and removes the entry. Until
then, the planning blocks every task that depends on it (step 5), and the
run goes on with everything else.

## Feedback

The user reports what they notice by hand with `/wcc:feedback`, which sorts
it with them and records it: bugs as `F` entries in ROADMAP.md, small things
in BACKLOG.md, results of manual checks in STATUS.md. While a run is going,
it writes them to FEEDBACK.md instead, so it does not edit files under the
run's feet.

So at the start, before picking the entry (step 1), read FEEDBACK.md if it
exists. Findings the user queues while this run is going wait for the next
run. Its findings are already sorted and confirmed by the user, so
record them without questions, the way `/wcc:feedback` does:

- check ok: remove the check from "Open manual checks" in STATUS.md.
- bug, and `minor, soon`: add it to the unchecked `F` entry in ROADMAP.md whose plan has no
  tasks yet, or insert `- [ ] F<n> Feedback fixes` before the next entry
  that has not started (after the last entry of the stage if none is left),
  and copy the finding with all its details into `plans/F<n>.md` under
  "Findings".
- other minor findings: BACKLOG.md with `(source: feedback)`.
- wish: "Open decisions" in STATUS.md, source `feedback` (see "Open
  decisions").

Then commit the changed planning documents and delete FEEDBACK.md. If the
run starts on a milestone branch (an interrupted milestone it resumes), do
this after that milestone's merge, before the report, so the base branch
gets the entry.

## Feedback milestones

A feedback milestone fixes bugs the user found by hand, and minor findings
the user did not want to wait for. It works on the
branch `wcc/f<n>` like a milestone, with these differences:

1. The plan `plans/F<n>.md` already holds the findings, recorded by
   `/wcc:feedback`. Resume as usual if it also has tasks.
2. Plan one task per finding, or one per group of findings with the same
   cause. A finding that turns out to need a change to SPEC.md or
   ARCHITECTURE.md becomes a wish: mark its task `[!]` with ` - wish, see
   /wcc:change` and treat it as a task that cannot be finished.
3. Give the `test-writer` the findings along with the criteria they
   concern: every finding that a test can show gets a test that fails now.
   A finding no test can show (a look, a feel) gets none; at the close it
   goes to "Open manual checks" with a test guide, under its criterion ID or
   as `F<n>.<k>` if it has none, so the user confirms the fix.
   A minor finding stays within SPEC.md and UI-SPEC.md: if the fix would
   change what they say, it is a wish (see 2).
4. Implement, review and close like a milestone (steps 7 to 9), with the
   documentation task if a fix changes what users or developers see.
   Delete the files in `feedback/` the plan names once it is closed.
   After a stage was tagged, the tag stays where it is.

`F` entries do not count as milestones for "Backlog milestone after", and
they do not reset that count.

## Backlog milestones

After closing a milestone, look at the checked entries of the current stage.
Let `k` be the value of `Backlog milestone after` in STATUS.md (3 if the line
is missing). Skip `F` entries when looking back. If `k` is not 0 and the
last `k` entries are all `M` entries, or
the stage has no open `M` entry left and its last entry is not a `B` entry,
insert `- [ ] B<n> Backlog after M<x>`
right after the milestone just closed. It is then picked like any other entry,
and the roadmap shows after an interruption whether it already ran.

A backlog milestone works on the branch `wcc/b<n>` like this:

1. Read BACKLOG.md. Drop items that are obsolete, contradict SPEC.md or
   ARCHITECTURE.md, or are worth less than the task they would cost (a
   matter of taste, a rare case without harm), with a one-line note.
2. Work off the remaining items like normal tasks: tests stay green, one
   review round, commit per item. Most items are small, so group them by
   component into tasks of a few items each, and group small tasks for one
   review as step 5 describes. Documentation gaps among them are
   collected into the documentation task; it runs if one exists or if an
   item changed behaviour users or developers see.
3. Items that would need a change to SPEC.md or ARCHITECTURE.md move from
   BACKLOG.md to "Open decisions" in STATUS.md (see there) and are listed in
   the report.
4. Archive before the merge: move every checked item of BACKLOG.md, done
   or dropped, also those checked off by `/wcc:change` or `/wcc:feedback`
   since the last backlog milestone, unchanged with its notes to
   `BACKLOG-DONE.md` under a new heading `## B<n>`, newest heading first.
   Create the file from `../idea/assets/BACKLOG-DONE.md` if it is missing.
   BACKLOG.md keeps only open items. Every backlog milestone reads it in
   full, and checked items with their notes otherwise grow to most of the
   file. No skill reads BACKLOG-DONE.md; it is for looking things up.
5. Close it like a milestone (step 9). Its full test suite always includes
   all end-to-end tests (see "End-to-end tests").

Findings from the review of a backlog milestone go back into BACKLOG.md for
the next one; they do not extend the current one.

## After the milestone

A run implements one roadmap entry and stops after its report. Do not start
the next entry, also when it is a backlog or feedback milestone you just
inserted. Every milestone reads the project files afresh anyway, so what
this one left in the context would only make every later step of the session
more expensive and the work less precise: each call sends the whole
conversation again. The next entry starts with the user's next
`/wcc:milestone`, best in a new session or after `/clear`.

Set STATUS.md before the report:

- entries of the current stage are still open: `IN_PROGRESS`, with the next
  entry named as not yet started under "Next", so the next run picks it up
  without questions.
- the current stage is complete: tag the base branch `wcc-stage-<n>`, unless
  that tag exists already (a feedback milestone after the stage end), and set
  `DONE`, or `MANUAL_CHECK` if manual criteria are still open. If "Open
  decisions" has entries, list them in the report and name `/wcc:change`
  under "Next" in STATUS.md: the next stage is planned there anyway.
- a task could not be finished: `BLOCKED`, as described above.
- the milestone waits for a manual check: `MANUAL_CHECK`, as described under
  "Manual criteria".

End the report with the next command: `/wcc:milestone` in a new session for
`IN_PROGRESS`, `/wcc:feedback` and then `/wcc:milestone` for
`MANUAL_CHECK`, `/wcc:milestone` for `BLOCKED`, `/wcc:change` for `DONE`.

Do not start a target vision item on your own. It needs a decision by the user,
which `/wcc:change` turns into the next stage.
