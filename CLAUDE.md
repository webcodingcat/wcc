# wcc: development notes for Claude Code

This repository is a plugin for Claude Code. It bundles a workflow that takes
a project from a raw idea to unattended, test-driven implementation.

This file is for Claude Code when someone wants to develop the plugin further
or adapt it to their own way of working. It explains purpose, structure and
the reasons behind the rules, so that changes do not break the workflow by
accident. The README is for the people who use the plugin.

## Language

- Everything in this repository is English: this file, skills, agents,
  README and the templates for planning documents in `skills/*/assets/`.
- The plugin does not set a conversation language, neither in the skills
  nor in the project `CLAUDE.md`. Claude talks to everyone in the language
  they write in or have configured locally. A rule in the project would
  force it on everyone who clones the repository. The configured language
  comes first in the skills: a skill called without text carries no
  language, and the English skill text and documents in the context then
  pulled the answer into a random one (version 0.17.1). The answer options in the
  skills are therefore written in English, with the instruction to offer
  them in the user's language. Everything that ends up in a project's
  repository is English: code, comments, repository docs and also the
  planning documents in `docs/wcc/`, so that others can work on the
  project. "Customising" says how to change that.
- The terms in the documents are keys that the skills and agents search for
  (`manual`, `Parallel agents`, `Backlog milestone after`, `Stops for: manual check`, `Cost of change`, `(working
  title)`, "Target vision", "Open manual checks", "Findings", `does not
  fit, see plan`, `F<n>`, `Start`/`Setup`/`Steps`/`Expected`,
  "Open decisions", `Q<n>`, `waits for Q<n>`, `In clarification`,
  `Context`/`Options`/`Recommendation`/`Affects`, "Documentation",
  `wcc version:` and so on). Whoever changes one of them in
  a template changes it in the skills and agents too.
- Talk to the person working on this repository in the language they use
  with you.

## What the workflow is for

The user hands over an idea. It is worked out, the stack is chosen, and the
project is then implemented milestone by milestone: each one started with a
single command, in parallel where possible, and without questions while it
runs. The user's attention goes
almost entirely into the first step.

```
/wcc:idea       interview                          -> IDEA.md (the user's document)
/wcc:spec       derive, check, summarise           -> SPEC.md, ARCHITECTURE.md, ROADMAP.md
/wcc:prepare    check the machine, set up project  -> git, scaffold, working build and test commands
/wcc:milestone  plan, test, build                  -> code, milestone by milestone
/wcc:feedback   show open checks, take findings    -> F milestone, BACKLOG.md or /wcc:change
/wcc:change     change request, next stage,        -> all documents updated consistently
                answers to open decisions
/wcc:status     state and next command             -> read only
/wcc:options    options for the runs               -> section "Options" in STATUS.md
/wcc:upgrade    after a plugin update              -> project CLAUDE.md and documents on the new version
/wcc:prototype  throwaway prototype (optional)     -> decisions/prototype-learnings.md
```

## Structure

```
.claude-plugin/plugin.json        manifest
.claude-plugin/marketplace.json   makes the repository its own marketplace
                                  (source "./", as the docs intend for this case)
skills/idea/                      interview; assets = scaffold for new projects
skills/spec/                      spec, architecture, roadmap; assets = templates
skills/prepare/                   machine check and project setup, repeatable
skills/milestone/                 unattended implementation of one milestone
skills/change/                    change requests and the next stage
skills/status/                    state and next command, read only
skills/feedback/                  take and sort findings from manual testing
skills/options/                   show and set the options of milestone runs
skills/upgrade/                   bring an existing project up to the installed version;
                                  references/migrations.md = catalogue per version
skills/prototype/                 optional throwaway prototype
agents/                           idea-gap-checker, researcher,
                                  feasibility-checker, spec-reviewer,
                                  test-writer, code-reviewer,
                                  doc-writer, milestone-verifier
```

The rules that must hold in every session of a project (no questions during
the implementation, tests, reviews, language) are not in the plugin itself.
A plugin cannot ship a CLAUDE.md that is always loaded. So `/wcc:idea`
copies the template `skills/idea/assets/project-CLAUDE.md` into the project.
The consequence: a change to that template reaches existing projects only
through `/wcc:upgrade` (see "Bringing existing projects along").

## Files in a project

The project `CLAUDE.md` lives in the project root, because Claude Code loads
it from there. All other files live in `docs/wcc/`. The dedicated folder
keeps the planning documents apart from the rest of the repository docs and
the project root free for the code.

| File | Content | Written by |
|------|---------|------------|
| `IDEA.md` | What and why: problem, goals, non-goals, target vision, success criterion | the user in the interview |
| `SPEC.md` | Stack with versions, accounts and secrets (names only), machine prerequisites (elevated rights too), requirements, acceptance criteria with verification type, end-user and developer documents with path and criteria | Claude |
| `ARCHITECTURE.md` | Components, interfaces, extension to the target vision | Claude |
| `UI-SPEC.md` | Binding rules for the user interface; only for projects with one | Claude |
| `ROADMAP.md` | Milestones per stage, backlog milestones, rough target vision | Claude |
| `STATUS.md` | Current state; first line `STATUS: <value>`; options of the runs; open manual checks and open decisions for the user | Claude |
| `DECISIONS.md` | Every decision and assumption taken without asking, every approval; per entry the cost of a later change | Claude |
| `BACKLOG.md` | Review findings that are not blockers and need no decision by the user; open items only after each backlog milestone | Claude |
| `BACKLOG-DONE.md` | Checked backlog items with their notes, moved there at the close of each backlog milestone; read by no skill | Claude |
| `FEEDBACK.md` | Queue: findings from `/wcc:feedback` reported while a milestone was running; exists only until the run records them | Claude |
| `feedback/` | Image files for findings a feedback milestone needs | Claude |
| `decisions/` | Results of clarification sessions | Claude |
| `EXPERIMENTS.md` | What the feasibility check tried: assumption, result, the decisive commands, consequences; each entry `valid` or `superseded` | Claude |
| `experiments/` | Working folder of the feasibility check; deleted after the check and listed in `.gitignore` | Claude |
| `plans/` | Detailed plan per milestone with research results and task check marks (`[x]` done, `[!]` cannot be finished), for Claude only | Claude |

Status values: `IDEA`, `IDEA_DONE`, `SPEC_REVIEW`, `READY`, `IN_PROGRESS`,
`MANUAL_CHECK`, `BLOCKED`, `DONE`.

| Status | Meaning | Continue with |
|--------|---------|---------------|
| `IDEA` | Interview going on | `/wcc:idea` |
| `IDEA_DONE` | IDEA.md is finished | `/wcc:spec` |
| `SPEC_REVIEW` | Summary waits for approval | `/wcc:spec` (shows it again) |
| `READY` | Approved, or a new stage planned | `/wcc:prepare`, then `/wcc:milestone` |
| `IN_PROGRESS` | Implementation running, interrupted, or between two milestones | `/wcc:milestone`, best in a new session |
| `MANUAL_CHECK` | Run waits for a manual check | `/wcc:milestone` (asks for the result) |
| `BLOCKED` | Run needs a decision | `/wcc:milestone` (asks for it) or `/wcc:change` |
| `DONE` | Current stage finished | `/wcc:change` for the next one |

## Design decisions and the problems behind them

Keep these when editing. Each one answers a concrete problem that comes up
when working with Claude Code without such rules.

- **The interview asks only what the user can answer, with options and a
  recommendation.** Interviews without this limit drift into technical
  details the user cannot say anything about.
- **Ask in the interview; assume and record during the implementation.**
  The difference is whether the user is present. A question in an
  unattended run stops it for hours.
- **The completeness of the interview is checked, not felt.** Mandatory
  sections plus the `idea-gap-checker`, because "Claude thinks it is done"
  is not a criterion.
- **Assumptions are presented for confirmation.** Confirming a list is
  faster than answering twenty questions, and nothing is decided invisibly.
- **Clarification sessions for questions that need a discussion** (the
  stack, for example). They run as child sessions in Claude Code Desktop so
  the long discussion does not fill the context of the interview. The
  signal for "done" is the result file in `decisions/`, not a message from
  the child session: a session in which the user writes themselves does not
  report back to the parent session reliably.
- **Target vision separate from the first version.** The architecture is
  designed for the target vision; code is written only for the current
  milestone. Projects that start small and grow need the right interfaces
  from the start.
- **Detailed plans are not shown to the user.** They are too detailed to
  read and would be approved blindly. The user gets a summary of at most ten
  lines.
- **One milestone per run, started by hand (version 0.25.0).** Up to
  0.24.0 a run chained the milestones of a stage in one session. Every
  milestone reads the project files afresh anyway, so what the earlier ones
  left in the context helped nothing, but every later call sent it again:
  the session grew with each milestone, used up the usage limits faster
  and worked less precisely once the context was long or compacted. The
  alternatives were rejected: a child session per milestone works only in
  Claude Code Desktop, and its reports and chained starts are not reliable
  there; a subagent per milestone cannot start the `test-writer` and the
  reviewers, because subagents start no subagents; an external loop around
  `claude -p` was not wanted. The price: the user starts every milestone,
  backlog and feedback milestones included, best in a new session or after
  `/clear`. A milestone itself still runs unattended. The option `Stop
  after each milestone` and the arguments `--once` and `--all` were
  removed with it.
- **The main session stays small.** Whatever reaches it stays in its
  context for the rest of the run. So build, test and lint output goes to
  files and only the summary and the failures are read, and every subagent
  returns a short result in a fixed form, without logs, code or an account
  of how it worked (section "Keeping the main session small" in
  `skills/milestone/SKILL.md`, last paragraph of the agents used there).
- **Reviews have a stopping rule.** Only blockers are fixed, the rest goes
  to BACKLOG.md, one round per task, the architecture is not up for debate.
  Reviewers without this limit always find something and hold everything
  up.
- **Tests come first, from a separate agent, from the spec.** Otherwise
  tests only confirm what the code does anyway, and "done" has no objective
  meaning.
- **Every acceptance criterion names its verification type** (auto,
  browser, manual). The share of `manual` decides how much can run
  unattended. It is large for games and small for APIs.
- **State lives in files, not in the context.** Every milestone reads the
  project files afresh, so runs can be interrupted and resumed. That is why
  backlog milestones are a line in the ROADMAP and open manual checks are in
  STATUS.md.
- **Questions are asked only when a skill is invoked.** At that moment the
  user is at the computer. So `/wcc:milestone` resolves `BLOCKED` and
  `MANUAL_CHECK` at the start and asks nothing afterwards.
- **Prerequisites are checked while the user is present.** `/wcc:prepare`
  finds missing tools before an unattended run fails on them, and can be
  repeated for new machines and fresh checkouts. The skill does not install
  system software itself.
- **Tests are committed before the implementation starts.** A worktree
  contains only what is committed. Uncommitted tests would not reach the
  parallel subagents.
- **A task is done when its own tests are green; a milestone when all are
  green.** The tests are written for the whole milestone, so those of the
  tasks still open necessarily fail.
- **Only the main session writes DECISIONS, BACKLOG, STATUS and ROADMAP.**
  Parallel worktrees changing the same file cause merge conflicts.
  Subagents report their decisions back.
- **Research comes before choosing the stack and before planning.** The
  model's memory of libraries is often one version behind. The `researcher`
  looks up versions, interfaces and pitfalls in the current documentation
  before something is chosen or used for the first time.
- **A task gets three repair attempts.** Without a limit, a run gets stuck
  on a single task.
- **Block late.** A task that cannot be finished does not end the run at
  once. Dependent tasks are skipped, all others finished, and only then
  `BLOCKED` follows. A stuck task thus costs the user one decision, not the
  whole night.
- **Repairing and adding is allowed, changing and replacing stops the run.**
  If every deviation stops it, the run halts on missing input validation.
  If none does, an agent rebuilds the architecture. Allowed is what breaks
  nothing other tasks and the tests build on; when in doubt, the run stops.
- **Claude never takes consequential decisions alone.** Data schema, save
  formats, public interfaces and authentication are expensive to change
  later. They are caught in three places: the spec must settle them for the
  first version and present them for approval, the planning of every
  milestone looks for leftovers before anything is built, and in a run an
  open one is a reason to stop. Stopping is better than guessing here, and
  should be rare thanks to the first two stages.
- **The milestone is checked as a whole.** Green tests do not prove that
  the code is wired in. The `milestone-verifier` looks for stubs and
  unwired code and believes no report.
- **Missing credentials are not a bug but a reason to stop.** They are
  named in the SPEC, `/wcc:prepare` checks them beforehand, and in a run
  they are never worked around with placeholders.
- **No agent obtains elevated rights itself.** If the feasibility check
  needs administrator rights, the `feasibility-checker` prepares the
  command and `/wcc:spec` has the user run it, who is present at that
  moment. If the project itself needs them, the SPEC says so,
  `/wcc:prepare` checks it, and in a run missing rights are a reason to
  stop. A run with administrator rights gives them to every agent; so the
  user decides this before the approval, not an agent in the middle of the
  night.
- **The feasibility check reports missing programs instead of working
  around them.** If a tool for an experiment is missing, the agent does not
  install it, nor does it silently replace the experiment with reading.
  `/wcc:spec` shows the user the list with install commands and repeats the
  check afterwards.
- **The feasibility check may write, but only in `docs/wcc/experiments/`.**
  An experiment that may not create anything checks nothing. The fixed
  folder keeps it apart from the project code.
- **What remains of an experiment is the result, not the code.** `/wcc:spec`
  summarises the experiments in EXPERIMENTS.md and deletes the folder. Only
  the insight is reused anyway; leftover code goes stale and gets built or
  committed by accident. So every entry contains the commands to repeat the
  check. The planning of every milestone reads the file so nothing is found
  out twice.
- **No falling back to similar package names.** A near miss may be a
  malicious package.
- **Git rules for parallel agents.** No `git add -A`, no `git stash`, no
  `git reset --hard`, one agent by default, at most five at the same time.
- **The user decides the parallelism.** Parallel agents use up the usage
  limits correspondingly faster, and a run that hits the limit stands
  still. `/wcc:options` or `/wcc:milestone --parallel <n>` (1 to 5) sets the
  upper bound. It is stored as `Parallel agents` in STATUS.md and not in the
  context, so it survives interruptions and holds for all following
  milestones. The default is 1 (since version 0.11.0, 3 before): a run
  should not use up the limits faster than the user explicitly chose.
- **Options live in the project, in STATUS.md.** Section "Options" with
  `Parallel agents`, `Backlog milestone after` and `Full e2e run`, set
  with `/wcc:options`. There they survive interruptions like the rest of
  the state, and `/wcc:milestone` reads them at the start of every run.
  Arguments of a call hold only for that run. Exception: `--parallel` is
  written back, as it was before the skill existed.
  Projects without the section get it with the first `/wcc:options` or
  `/wcc:upgrade`; until then the defaults apply, plus a `Parallel agents`
  line under "Current state".
- **Expensive end-to-end tests do not necessarily run in full at every
  milestone.** End-to-end tests against the real environment grow with
  every milestone; in one project they already took almost half an hour per
  close after M3. With `Full e2e run: backlog and stage end` an ordinary
  milestone runs them only for its own criteria and changed test files, all
  of them at every backlog milestone and before the tag of a stage. The
  price: a regression in an older flow shows up only there. The default
  stays `every milestone`. End-to-end tests are what the project
  `CLAUDE.md` starts with a separate command; `/wcc:prepare` separates that
  command. The cause of the long run time there was a full setup per test;
  so the `test-writer` requires tests to share an expensive setup when they
  only use it. Version 0.19.0 closed two gaps that made the option useless
  in that project: runs knew which tests to select but not how, and ran the
  plain command, so every close ran all of them; and the shared setup had
  led to a few large tests whose names bundled the IDs of dozens of
  criteria, so nearly every milestone selected nearly everything. Now the
  milestone skill turns the selection into the filter of the test framework
  and records the command, and the `test-writer` gives every criterion a
  subtest of its own under a group named after the component.
- **Documentation is asked for in the interview and written per milestone
  (version 0.13.0).** Without a rule, end-user and developer documentation
  is not written at all or only at the end, when it has to describe
  everything at once. The interview asks who needs which documentation,
  and the SPEC lists documents (`D<n>`) in its section "Documentation" with
  a path outside `docs/wcc/` and checkable criteria. The planning documents
  are not documentation: they hold decisions and check marks for Claude,
  not explanations for people. The last task of every milestone is
  `Documentation`; it runs after the merge through the `doc-writer`,
  because documentation describes the wired-in result and parallel tasks
  would collide on the same documents. Unlike the `test-writer`, the
  `doc-writer` reads the code and runs every example. The documentation
  criteria hold at every close and are checked against what was built, not
  against the whole spec. The `milestone-verifier` checks "documented" and
  "true", at the end of a stage for the whole stage.
- **The project name may come later (version 0.14.0).** Ideas often have no
  name yet, but the plugin needs a folder. The interview asks for the name
  at the end and allows a working title (`# IDEA: <name> (working
  title)`). Renaming happens between `/wcc:idea` and `/wcc:spec`, because
  nothing depends on the path until `/wcc:prepare`; the skill cannot rename
  the folder itself, because the session runs inside it. `/wcc:prepare`
  asks once more before `git init` if the name is open or does not match the
  folder, and never again afterwards.
- **One branch per milestone.** The base branch contains only finished
  milestones, also when a run ends blocked.
- **The plan carries a check mark per task.** After an interruption it is
  clear where to continue, without planning again or rewriting tests.
- **User interfaces are specified up front.** Agents that build views in
  parallel otherwise each invent their own look.
- **Manual checks stop the run only if later milestones depend on them.**
  The milestone then stays open until the user has reported the result. All
  other manual criteria collect in STATUS.md.
- **The user reports findings from manual testing with `/wcc:feedback`
  (version 0.15.0).** Before, only the result of a manual check could be
  reported, and only when `/wcc:milestone` asked for it; anything else the
  user found while trying the project had no way in. A skill fits better
  than a file to fill in: the user says in their own words what they
  noticed, with a screenshot too, and is there to answer questions at that
  moment. The skill sorts every finding (check ok, bug, minor, wish) and
  presents the list as one question for confirmation. Bugs become a
  feedback milestone `F<n>`, which like every milestone first writes a
  failing test; minor findings go to BACKLOG.md, wishes straight through
  `/wcc:change`. The user can pull a minor finding forward when confirming
  (`minor, soon`); it then goes into the feedback milestone instead of
  waiting for the next backlog milestone. What no test can show goes back
  to the user as a manual check afterwards. The run cannot see images from
  the conversation, so the skill describes every screenshot in words in the
  plan. If a milestone is running in the same checkout, the skill does not
  write to the ROADMAP and plans but to the queue FEEDBACK.md; the run
  records it before the next milestone without asking, because the user has
  already confirmed.
- **Every open manual check has a test guide.** A short note in STATUS.md
  was not enough to test without knowing the code. The run adds start,
  setup, steps and expected result when it closes the milestone, while it
  still has the code in front of it. `/wcc:feedback` without arguments shows
  the checks with their guides and takes the results right away. A separate
  skill would be one more command; showing and reporting belong together.
- **`/wcc:upgrade` brings existing projects along (version 0.16.0).**
  Before, this was done with prompts while developing the plugin, anew for
  every project and without a fixed procedure. Projects are started all the
  time, and each one needs an upgrade again after later versions, also
  projects nobody knows while developing the plugin. So the skill knows no
  project, only two sources: the catalogue
  `skills/upgrade/references/migrations.md` with a detection rule per entry,
  and the comparison with the current templates. The whole catalogue is
  always checked, not only from the project's version on, because projects
  may have been brought along by hand or only in part. The version is the
  line `wcc version:` in the project `CLAUDE.md`; older projects lack it and
  count as "before 0.16.0". The skill changes the form, never the
  decisions: what needs an answer from the user (such as the section
  "Documentation") it does not create, but names the skill that asks for
  it. It presents one list and asks once, because the user is present when
  invoking it. It stops while a milestone is running, because it writes the
  same files. `/wcc:status` and `/wcc:milestone` point out a project that is
  behind.
- **Open decisions wait in STATUS.md, not in the backlog (version
  0.17.0).** Before, findings that would change SPEC.md or ARCHITECTURE.md
  stayed in BACKLOG.md marked `needs decision`. The user learned of them
  only from the report of a backlog milestone, which scrolls away in an
  unattended run, or by calling `/wcc:status`; nothing asked, and the items
  had no question to answer and often mixed several points. Now the run
  writes each one as a question with context, options and a recommendation
  (`Q<n>`) under "Open decisions" in STATUS.md, where the user looks for
  what waits for them. Writing down a recommendation is not deciding, so
  the rule "no questions during the run" holds. Reviewers return them as a
  category of their own. Every report names the new ones, `/wcc:milestone`
  asks once at the start whether to decide them first, and `/wcc:change`
  without arguments takes the answers, because an answer changes the
  documents. The planning blocks tasks that depend on an open entry, so
  the run never builds on something undecided. `/wcc:change` asks like the
  interview, with "I don't know" and "I need more details" for every
  question: open decisions come up in a run, and the user meets them
  without the context they arose in, so they often need a discussion. That
  runs in a clarification session as in `/wcc:idea`; the entry gets the
  line `In clarification` with its result file, which is again the signal
  for "done".
- **Checked backlog items move to BACKLOG-DONE.md (version 0.18.0).**
  Items were checked off with notes but never removed. In one project after
  four backlog milestones, 180 of 250 items were checked and made up about
  three quarters of a file of 1800 lines, which every backlog milestone
  reads in full. Deleting them would lose the notes, some of which hold
  findings such as the behaviour of a tool version. So a backlog milestone
  moves every checked item unchanged to BACKLOG-DONE.md under `## B<n>`
  before it merges; no skill reads that file. It happens there because the
  backlog milestone is the only step that reads the whole backlog, and the
  main session writes BACKLOG.md then anyway.
- **Large planning documents are read by their headings (version
  0.20.0).** Every milestone read SPEC.md, ARCHITECTURE.md and DECISIONS.md
  in full, and every task review SPEC.md and ARCHITECTURE.md. In one project
  after 17 milestones these files had grown to about 400 KB together
  (DECISIONS.md alone from 89 to 204 KB in a day), and the milestones took
  longer each time: 25 minutes for M8, two and a half hours for M17. Now a
  file of more than 300 lines is read by its headings: the sections the
  milestone covers, the ones that hold for every milestone, and the
  DECISIONS.md entries whose title concerns it plus every one with
  `Cost of change: high`. The main session passes the names of the
  sections to `test-writer`, `code-reviewer` and `milestone-verifier`,
  which look up more by heading when needed. When in doubt a section is
  read: overlooking a decision costs more than reading it. Archiving old
  decisions like BACKLOG-DONE.md was rejected: unlike a checked backlog
  item, a decision stays in force. Instead entries stay short, with
  telling titles, because those are what is searched.
- **Shorter closes and less backlog (version 0.21.0).** In the same
  project, the close of a milestone took over an hour, almost all of it
  the end-to-end run, and backlog milestones three hours each. Three
  changes: the `milestone-verifier` runs while the full test suite runs in
  the background, because it only reads code; its fixes wait until the
  suite has ended, so they do not change the code under the running tests.
  Reviewers return at most five backlog items each, only what is worth a
  task, nothing that is a matter of taste, and a backlog milestone drops
  items worth less than their task. ARCHITECTURE.md holds what other
  components build on; how a component works inside goes into code
  comments, because the file is read by every milestone and had grown to
  130 KB of mostly such details.
- **Narrower checks per task, shared reviews for small tasks (version
  0.22.0).** A task is checked with its own tests, the other tests and the
  linter of the packages it changed, and the build; the whole unit suite
  and the linter over the whole project run once at the close, next to the
  end-to-end tests. Moving all checks to the end was rejected: a task with
  failing tests would reach the review and the merge, dependent tasks would
  build on it, and the repair attempts need the task's own tests as their
  signal. One review for the whole milestone was rejected too: over a
  change of eleven tasks the reviewer's context fills and it overlooks
  more, and a blocker found late has spread into the tasks built on it.
  Instead the plan groups up to three small tasks of one component, about
  300 changed lines together, for one review (`Review: with T<n>`); tasks
  in parallel worktrees and tasks that change an interface others build on
  are reviewed alone. Backlog milestones, with mostly small items, gain
  the most. `/wcc:prepare` names the narrower form of the commands; in
  older projects the run derives it from the tool.
- **Parallel test-writers (version 0.23.0).** Writing the tests took one
  `test-writer` over twenty minutes per milestone in the same project.
  With `Parallel agents` above 1, the criteria are split by component into
  groups with their own test files, each with a writer of its own, within
  the same limit as the implementation. The writers share the checkout
  without worktrees, because the tests are committed only afterwards, so
  each writes only its own files; a shared helper or the end-to-end harness
  belongs to one group, the others report what they need there. They run
  only their fast tests: end-to-end runs at the same time would collide in
  the real environment, so the main session runs the new end-to-end tests
  once when all are done. Small milestones keep one writer, since every
  writer reads its sections first.
- **ARCHITECTURE.md is checked against the code (version 0.24.0).** When
  the same project slimmed down its ARCHITECTURE.md, the cleanup found
  about 25 places where it said something other than the code: option
  fields and return values added by tasks, timeouts changed by review
  fixes, a rule changed by `/wcc:change` in one of two sections that
  stated it. Nothing compared the document with the code: the
  `code-reviewer` reports only what breaks an interface, an addition is
  allowed, and the main session learned of additions only when a subagent
  mentioned them. Since 0.20.0 test-writers and reviewers rely on the
  sections they are given, so a stale section turns into wrong tests. Now
  subagents list every change to what other components use, additions
  included; the main session and `/wcc:change` search the whole file for
  what changed; and the `milestone-verifier` compares the sections of the
  changed components with the code (check 7). Where the code is backed by
  a decision, the plan or an allowed addition, the main session updates
  the document at the close, also in a backlog milestone, because catching
  up is not a change of the architecture; a break without explanation is a
  blocker, an unclear case an open decision.

## Bringing existing projects along

Whoever changes something in the plugin that affects an existing project
adds it in the same change to `skills/upgrade/references/migrations.md`:
the version, how to detect that it is missing, what `/wcc:upgrade` changes
and which skill has to run afterwards. A project is affected when a skill
or agent newly expects, renames or reads in a different form a section, a
line or a term in the project files. Rules that live only in
`skills/idea/assets/project-CLAUDE.md` need no entry; the skill compares the
project `CLAUDE.md` with the template anyway. An entry must work on every
project, not only on the ones you know: the detection rule checks the
content of the files, never project names or assumptions about the
project's history.

## Customising

Typical changes and where to make them:

| Wish | Where |
|------|-------|
| Fixed conversation language | Best locally, in your own `~/.claude/CLAUDE.md`. In the plugin: the paragraph starting "Talk to the user in the language they have configured" in every `SKILL.md` and the section "Language" in `skills/idea/assets/project-CLAUDE.md` |
| Different document language | Language rule in every `SKILL.md` and in `skills/idea/assets/project-CLAUDE.md`, templates in `assets/`, return language in `agents/*.md`, and every term from the templates that skills and agents quote literally |
| Rules for the implementation (questions, stop conditions) | `skills/idea/assets/project-CLAUDE.md` |
| Interview questions and mandatory sections | `skills/idea/SKILL.md`, `skills/idea/assets/IDEA.md` |
| Another project type | one section each in `skills/idea/references/project-types.md` and `skills/spec/references/project-types.md` |
| Stricter or looser reviews | Blocker definition in `agents/code-reviewer.md` and `agents/spec-reviewer.md`; the limit of backlog items in `agents/code-reviewer.md` and `agents/milestone-verifier.md` |
| Your own review agents | replace `agents/code-reviewer.md`; they need the same split into blocker and backlog |
| Drop the approval after the spec | Step 7 in `skills/spec/SKILL.md` |
| Criteria for choosing the stack | Step 1 in `skills/spec/SKILL.md` |
| Different folder for the planning documents | The sentence "All planning documents live in …" in every `SKILL.md`, `skills/idea/assets/project-CLAUDE.md`, the path in `agents/*.md`, README |
| More machine checks | Step 1 in `skills/prepare/SKILL.md` |
| What is allowed without stopping, what counts as consequential | Section "During a milestone run" in `skills/idea/assets/project-CLAUDE.md`, explanation in `skills/idea/assets/DECISIONS.md` |
| Number of repair attempts, reasons to stop | `skills/idea/assets/project-CLAUDE.md` and section "Tasks that cannot be finished" in `skills/milestone/SKILL.md` |
| Number of parallel agents, backlog rhythm in a project | `/wcc:options` or section "Options" in STATUS.md |
| Another option | Table in `skills/options/SKILL.md`, section "Options" in `skills/milestone/SKILL.md` and the place that reads it, template `skills/idea/assets/STATUS.md`, README |
| Default value and limits of an option | `skills/options/SKILL.md`, section "Options" in `skills/milestone/SKILL.md`, template `skills/idea/assets/STATUS.md`, README |
| Branch and tag names | `skills/milestone/SKILL.md` |
| Scope of the final check | `agents/milestone-verifier.md` |
| Kind and scope of the documentation | Interview question in `skills/idea/SKILL.md` and recommendations in `skills/idea/references/project-types.md`, section "Documentation" in `skills/idea/assets/IDEA.md` and `skills/spec/assets/SPEC.md`, rules in `skills/spec/SKILL.md` and `skills/spec/references/project-types.md`, writing rules in `agents/doc-writer.md`, check in `agents/milestone-verifier.md` |
| Model of an agent | `model` in the header of `agents/*.md` |
| What an upgrade changes in existing projects | `skills/upgrade/references/migrations.md`; the procedure in `skills/upgrade/SKILL.md` |
| Handling of findings from manual testing | `skills/feedback/SKILL.md`, sections "Feedback" and "Feedback milestones" in `skills/milestone/SKILL.md`, README |
| Which parts of the planning documents a milestone and its agents read | Section "Reading the planning documents" in `skills/milestone/SKILL.md`, the opening paragraph of `agents/test-writer.md`, `agents/code-reviewer.md` and `agents/milestone-verifier.md`, step 1 in `skills/feedback/SKILL.md` |
| Rule for backlog milestones | Section "Backlog milestones" in `skills/milestone/SKILL.md`; the rhythm is the option `Backlog milestone after` |
| What counts as an open decision, its form, when it is asked | Section "Open decisions" in `skills/milestone/SKILL.md`, the categories in `agents/code-reviewer.md` and `agents/milestone-verifier.md`, sections "Open decisions", "Asking" and "Clarification sessions" in `skills/change/SKILL.md`, template `skills/idea/assets/STATUS.md` |
| How clarification sessions work | Step 2 in `skills/idea/SKILL.md` and section "Clarification sessions" in `skills/change/SKILL.md`; change both |

Whoever changes a rule that appears in several places changes all of them.
The rules in `project-CLAUDE.md` and in the agents must agree with each
other.

## Testing changes

Start Claude Code with the plugin from this directory:

```
claude --plugin-dir .
```

Manifest and marketplace file can be checked with `claude plugin validate
.`. Then try the skills as `/wcc:idea`, `/wcc:spec` and so on in an empty
test project. After editing a skill or agent, reload the plugins or restart
the session.

Two test cases cover most of it: a small project with fixed rules (a
Tetris-like game, for example) and a project that starts small and has a
large target vision (a re-implementation of an existing API with many
resource types, for example).

## Conventions when editing

- The `description` of a skill decides when it is triggered. Keep the
  trigger phrases. They are English; the skills also trigger on input in
  other languages, because the description is matched by meaning, not by
  wording.
- Explain the reason behind an instruction instead of repeating it with
  emphasis.
- Keep every `SKILL.md` short. Material that applies to only one project
  type belongs in `references/`.
- Agents that only judge have read access only (`tools: Read, Grep,
  Glob`). Exception: `code-reviewer` also has `Bash`, to read the changes
  with git and run the tests. Its prompt forbids any change to files. The
  `feasibility-checker` does not only judge, it tries things out: it has
  write access, and its prompt restricts it to `docs/wcc/experiments/`.
- Raise `version` in `plugin.json` with every change that changes the
  behaviour. If the change needs something in existing projects, it comes
  with an entry in `skills/upgrade/references/migrations.md`.
- Adjust author and owner in `plugin.json` and `marketplace.json` when
  passing the plugin on under your own name.

## Decisions taken

These points were open and are decided. Entries marked "assumption" were
taken over from a recommendation because there was no preference of one's
own. They can be changed without consequences for the rest.

- **The approval after the spec stays.** It is the only fixed stopping
  point: Claude waits for the OK on the summary before the implementation
  starts.
- **Planning documents are English (version 0.10.0).** Up to 0.9.0 they
  were German, because the user reads them. It was decided otherwise
  because others may work on a project too, and a repository in two
  languages shuts them out. The price: the user reads IDEA.md and the other
  documents in English. For the same reason the plugin no longer sets a
  conversation language: the conversation, the summary for approval and the
  reports follow the user's language. Projects with German documents no
  longer fit the skills, because these search for the English terms;
  `/wcc:upgrade` translates them.
- **The plugin repository is English too (version 0.16.0).** Up to 0.15.0
  this file was German. Like the projects, the plugin is meant to be
  developed by others as well.
- **Backlog (assumption):** a backlog milestone runs after every third
  milestone and at the end of a stage. Since version 0.11.0 the rhythm is
  the option `Backlog milestone after`. It is entered as a line `B<n>` in
  the ROADMAP. Governed by `skills/milestone/SKILL.md`.
- **Storage in `docs/wcc/`.** Comparable tools (Spec Kit, Kiro, GSD,
  OpenSpec, Agent OS, BMAD, Superpowers) all use their own folder. Visible
  rather than hidden, because the user reads IDEA.md themselves.
- **`/wcc:prepare` as a skill of its own.** Machine check and project
  scaffold sit in one place instead of being spread over idea, spec and
  milestone.
- **Clarification sessions in the interview and in `/wcc:change`.**
  `/wcc:spec` starts none of its own but reads `docs/wcc/decisions/`; the
  stack discussion belongs in the interview. `/wcc:change` has them since
  version 0.17.0, for open decisions and for questions a change raises.
- **Stages.** After `DONE`, `/wcc:change` plans an item of the target vision
  as the next stage with its own section in the ROADMAP.
- **Taken over from the comparison with "Get Shit Done" (version 0.4.0):**
  research, three repair attempts, final check, credentials as a reason to
  stop, package rule, git rules, task check marks, `/wcc:status`, UI-SPEC,
  one branch per milestone and one tag per stage.
- **Protecting `.env` with a deny rule instead of a hook.** `/wcc:prepare`
  adds `Read(./.env)` to the project's `.claude/settings.json`. A hook
  would need a script that runs on every operating system, and would
  disturb every read if it failed.
- **Cheaper model (assumption):** `spec-reviewer` and `researcher` run with
  `model: sonnet`, because they work through a fixed list or look things
  up. All other agents inherit the session's model.
- **Postponed:** a short path for single tasks and bug fixes after `DONE`,
  and support for existing projects that were not started with wcc. Both
  come when they are needed.
- **Throwaway prototype (assumption):** a separate, optional skill
  `/wcc:prototype`. The interview recommends it for games whose rules are
  still unclear.
- **No `/goal` (version 0.25.0):** a goal such as "STATUS.md says DONE"
  would chain the milestones in one session again, which this version
  removed. The tip is gone from the README.
- **marketplace.json** uses the source `"./"`. That is the form the
  documentation intends for a plugin published from its own repository.

## Open points

These points can only be settled in a running Claude Code installation.

1. **Session tools.** The idea and change skills use
   `mcp__ccd_session__start_session`; the tool exists in Claude Code
   Desktop. Not tried yet is whether the child session starts at once or
   first shows the user a card to click. The skill covers both cases.
   Observed: the report back to the parent session does not always arrive;
   since version 0.6.1 the skill no longer relies on it.
2. **Only part of it has run.** In one project, `/wcc:idea`, `/wcc:spec`,
   `/wcc:prepare` and the first milestone with `/wcc:milestone --once` ran
   without notable problems. `--parallel <n>` also ran in a run with
   `--once` and worked. Not yet run: `/wcc:change`, `/wcc:status`,
   `/wcc:options`, `/wcc:prototype`, a backlog milestone and the close of
   a stage. Also not yet run: the fixed result form of subagents and the
   run that stops after one milestone (since version 0.25.0). Also not yet run:
   the documentation task with the `doc-writer` (since version 0.13.0),
   `/wcc:feedback` with a feedback milestone (since version 0.15.0) and
   `/wcc:upgrade` (since version 0.16.0), the open decisions (since
   version 0.17.0) and the archiving to BACKLOG-DONE.md (since version
   0.18.0), and the filtered end-to-end run with subtests per criterion
   (since version 0.19.0), and reading the planning documents by their
   headings (since version 0.20.0), and the verifier running alongside the
   test suite (since version 0.21.0), the narrower checks per task and the
   shared reviews (since version 0.22.0), parallel test-writers (since
   version 0.23.0), and the check of ARCHITECTURE.md against the code
   (since version 0.24.0). The version 0.12.0 given for the
   option `Full e2e run` in the migration catalogue is inferred, not
   confirmed. `claude plugin validate .` has not been run since version
   0.3.0.
