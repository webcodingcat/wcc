# wcc: development notes for Claude Code

This repository is a plugin for Claude Code. It takes a project from a raw
idea to tested code, phase by phase. This file is for whoever develops the
plugin: what it is for, how it is built, and why the rules are as they are,
so changes do not break the workflow by accident. The README is for the
people who use it.

## Words

The words wcc uses:

| Word | Meaning |
|------|---------|
| Milestone | A version of the project (`## Milestone <n>` in ROADMAP.md). Ends with all end-to-end tests and the tag `wcc-milestone-<n>`. |
| Phase | A slice of a milestone that runs and can be checked; `/wcc:phase` builds one. IDs `01`, `02`, …, `F<n>` for feedback, `B<n>` for backlog phases. Folder `phases/<id>/`, branch `wcc/phase-<id>`. |
| Plan | One run of the `implementer` for one component inside a phase (`03-02`), one to three tasks, a wave, a size. |
| Task | A step inside a plan. |
| Wave | Plans without dependencies on each other; at most two agents run at the same time. |
| Requirement | One checkable capability in REQUIREMENTS.md (`AUTH-03`) with its test level. |
| Success criteria | Two to five observable results of a phase in ROADMAP.md. |
| Test map | `TESTS.md` of a phase: one row per promise of a requirement. |

## Language

- Everything in this repository is English: this file, skills, agents,
  README and the templates in `skills/*/assets/`.
- The plugin does not set a conversation language. Claude talks to everyone
  in the language they write in or have configured; a rule in a project
  would force one on everyone who clones it. Because a skill called without
  text carries no language and the English skill text would pull the answer
  into English, every skill says to use the configured language first, and
  quoted answer options are offered in the user's language. Results of
  agents and messages from other sessions arrive as messages in English;
  in the reference project's seventh phase the main session took them for
  the user's language and reported in English from the first test-writer
  on, despite a configured language. The skills that start agents or
  sessions say that these do not change the user's language.
- Everything that ends up in a project's repository is English, the
  planning documents included, so others can work on the project.
- Terms in the templates are keys the skills and agents search for:
  `STATUS:` and its values, `## Milestone <n>`, "Later", "Out of scope",
  "Coverage", the requirement IDs `<CAT>-<NN>`, the test levels
  `unit`/`e2e`/`none`/`manual`, `DOC`, "Success criteria", "Requirements:",
  `Stops for: manual check`, `Wave:`, `Depends on:`, `Review:`, `Size:`,
  `Adjusts:`, `Tests:`, `TESTS.md`, `CHECKS.md`, `CONTEXT.md`, `Result:`,
  `does not fit, see plan`, "Findings", "Findings from manual testing",
  "Items", "Close", "In clarification", `F<n>`, `B<n>`,
  `Q<n>`, `waits for Q<n>`, `In clarification`,
  `Context`/`Options`/`Recommendation`/`Affects`, "Open manual checks",
  "Open decisions", `Start`/`Setup`/`Steps`/`Expected`, `Cost of change`,
  `Type`, `(working title)`, "Documentation", `D<n>`, "References", the backlog
  severities `security`/`data`/`bug`/`later`/`minor`, `wcc version:`,
  `Replaces:`.
  Whoever renames one changes it everywhere.
- Talk to the person working on this repository in their language.

## The workflow

```
/wcc:idea       interview                          -> IDEA.md (the user's document)
/wcc:spec       derive, check, summarise           -> REQUIREMENTS, SPEC, ARCHITECTURE, ROADMAP
/wcc:prepare    check the machine, set up project  -> git, scaffold, working build and test commands
/wcc:phase      discuss, plan, test, build         -> one phase
/wcc:report     guided manual checks, bugs and     -> waiting phase, F phase or BACKLOG.md
                wishes about what is built
/wcc:change     new ideas, next milestone,         -> all documents updated consistently
                open decisions, backlog items
/wcc:status     state and next command             -> read only
/wcc:upgrade    after a plugin update              -> project files on the new version
/wcc:prototype  throwaway prototype (optional)     -> decisions/prototype-learnings.md
```

The user's attention goes into the interview, the approval of the spec and
the start of each phase. A phase may ask at its start ("Discuss") and then
runs without questions.

## Structure

```
.claude-plugin/plugin.json        manifest
.claude-plugin/marketplace.json   makes the repository its own marketplace (source "./")
skills/idea/                      interview; assets = scaffold and templates for new projects
skills/spec/                      requirements, spec, architecture, roadmap; assets = templates
skills/prepare/                   machine check and project setup, repeatable
skills/phase/                     one phase; references/ = e2e, backlog, feedback,
                                  read only when they apply
skills/change/                    changes, next milestone, open decisions, backlog
skills/status/                    state and next command, read only
skills/report/                    guided manual checks and findings
skills/upgrade/                   bring a project up to the installed version;
                                  references/migrations.md = catalogue per version
skills/prototype/                 optional throwaway prototype
agents/                           idea-gap-checker, researcher, feasibility-checker,
                                  spec-reviewer, planner, test-writer, implementer,
                                  code-reviewer, doc-writer, verifier
```

A plugin cannot ship a CLAUDE.md that is always loaded, so `/wcc:idea`
copies `skills/idea/assets/project-CLAUDE.md` into the project. A change to
that template reaches existing projects only through `/wcc:upgrade`.

## Files in a project

The project `CLAUDE.md` lives in the root, where Claude Code loads it.
Everything else lives in the hidden folder `.project/`, apart from the
repository docs and the code. Grep and Glob of Claude Code search hidden
folders; a plain `rg` needs `--hidden`, which the project CLAUDE.md says.

| File | Content | Written by |
|------|---------|------------|
| `IDEA.md` | What and why: problem, goals, non-goals, target vision, success criterion | the user in the interview |
| `REQUIREMENTS.md` | One line per requirement with ID, goal and test level, per milestone; "Later", "Out of scope", "Coverage" | Claude |
| `SPEC.md` | Stack with versions, references the project must match (path of the copy), accounts and secrets (names only), machine prerequisites, documentation `D<n>` | Claude |
| `ARCHITECTURE.md` | Components, interfaces, extension to the target vision | Claude |
| `UI-SPEC.md` | Binding rules for the user interface, if there is one | Claude |
| `ROADMAP.md` | Milestones with their phases, requirements and success criteria; finished phases one line each; "Later" | Claude |
| `ROADMAP-DONE.md` | Full entries of finished phases; read by no skill | Claude |
| `STATUS.md` | First line `STATUS: <value>`; current state in short lines; open manual checks (one line each); open decisions | Claude |
| `DECISIONS.md` | Every decision and assumption taken without asking, every approval, with the cost of a later change, as long as it holds | Claude |
| `DECISIONS-DONE.md` | Entries a later entry revoked or changed; read by no skill | Claude |
| `BACKLOG.md` | Parking lot: open items with severity and consequence | Claude |
| `BACKLOG-DONE.md` | Items worked off or dropped; read by no skill | Claude |
| `FEEDBACK.md` | Queue of findings reported while a phase was running | Claude |
| `research/EXPERIMENTS.md` | What the feasibility check tried, with the commands to repeat it; `valid` or `superseded` | Claude |
| `research/experiments/` | Scratch folder of the feasibility check; deleted afterwards, in `.gitignore` | Claude |
| `decisions/` | Results of clarification sessions | Claude |
| `phases/<id>/CONTEXT.md` | What the user decided at the start of the phase | Claude |
| `phases/<id>/PLAN.md` | Shared "Interfaces"; plans with wave, size, `Reads:`, tasks and check marks; findings or items for F and B phases; "Close" | Claude |
| `phases/<id>/TESTS.md` | Test map | Claude |
| `phases/<id>/RESEARCH.md` | What the researcher found, with sources | Claude |
| `phases/<id>/CHECKS.md` | Manual checks with guide and `Result:` | Claude |
| `phases/<id>/feedback/` | Images for findings of a feedback phase | Claude |

| Status | Meaning | Continue with |
|--------|---------|---------------|
| `IDEA` | Interview going on | `/wcc:idea` |
| `IDEA_DONE` | IDEA.md is finished | `/wcc:spec` |
| `SPEC_REVIEW` | Summary waits for approval | `/wcc:spec` |
| `READY` | Approved, or a new milestone planned | `/wcc:prepare`, then `/wcc:phase` |
| `IN_PROGRESS` | Between two phases, or a phase interrupted | `/wcc:phase`, best in a new session |
| `MANUAL_CHECK` | A phase waits for a manual check | `/wcc:report`, then `/wcc:phase` |
| `BLOCKED` | A plan needs a decision or a step of the user | `/wcc:phase` or `/wcc:change` |
| `DONE` | Current milestone finished | `/wcc:change` for the next one |

## Design principles

Keep these when editing. Each answers a problem that comes up when Claude
Code works on a project without such rules.

### Asking and deciding

- **The interview asks only what the user can answer**, with two to four
  options, a recommendation, "I don't know" and "I need more details".
- **Ask while the user is there; decide and record when not.** The
  interview, the spec approval, the start of a phase, `/wcc:report` and
  `/wcc:change` may ask. Inside a phase a question would stop it for hours,
  so it decides, records in DECISIONS.md with the cost of a later change,
  and blocks only where it must.
- **Each phase starts with "Discuss"**: Claude reads the requirements and the code, presents
  how it would build what they leave open as a list of assumptions to
  confirm, and asks only the open decisions of the phase and real grey
  areas, recorded in `CONTEXT.md`. Confirming a list is faster than
  answering questions, and the user is there anyway because they started
  the phase. The questions come after the list, each on its own: in the
  reference project they stood in the same message as the assumptions, the
  user answered the list with "fits" and three questions were taken as
  answered with their recommendation. Research runs while the user reads
  the list, and the questions come after it: the planner has no web
  access, and in the reference project's fifth phase it asked how a guest
  takes over network changes, with options from memory, although the
  answer was a fact about the original cloud that research would have
  found. What depends on facts outside the project is looked up, not
  asked; the questions wait about as long as the research takes. The
  turn ends with the list: in the reference project's seventh phase the
  question menu followed the list in the same turn, before the research
  was done, and the user's reply to the list had to break into the menu.
  In the eighteenth phase the research finished before the user replied,
  and the main session summarised it itself, with the unproven error
  codes as "here I would use" in prose: decisions the user would have
  passed over. What the research changes comes from the planner, after
  the reply, as changed assumptions and questions.
- **Assumptions are decisions, facts come from a reference.** In the
  reference project's sixth phase the planner's list named request fields
  and error codes of the original API from memory, some wrong, although
  the project kept a frozen copy of the API description; the user checked
  facts that were not theirs to confirm. So SPEC.md names under
  "References" what the project must match and where its copy lies, the
  planner reads the part of the phase there, and details of an outside
  system are no assumptions: they come from the reference or go to
  research.
- **Questions work without menus.** Where the AskUserQuestion tool is
  missing or its menus do not show (a remote session), the skills ask in
  text with numbered options, without a setting.
- **Completeness is checked, not felt.** Mandatory sections plus the
  `idea-gap-checker`, which sketches an architecture from IDEA.md alone and
  lists what it had to guess.
- **Assumptions are presented as a list to confirm.**
- **Long discussions run in clarification sessions** (child sessions in
  Claude Code Desktop). The signal for "done" is the result file in
  `decisions/`, because a session the user talks to does not report back
  reliably.
- **Consequential decisions are never taken alone.** Data schema, formats,
  public interfaces and authentication are settled in the spec, looked for
  again at the start of each phase, and a reason to stop inside one.
- **Open decisions wait in STATUS.md**, one question each with context,
  options and a recommendation. A bare list of blockers would lose the
  options and the recommendation, and most open decisions arise while
  nobody watches and are answered later without their context. The next
  phase asks those that concern it; `/wcc:change` takes answers any time.

### Planning

- **Requirements are atomic.** One line, one checkable capability, an ID
  that never changes, a test level. Requirements that bundled a dozen
  promises in one criterion pulled the same detail into tests, reviews and
  backlog. REQUIREMENTS.md also holds "Later", "Out of scope" and a
  coverage table.
- **Phases have success criteria**: two to five results a user can observe.
  They guide the planning, the verifier and the manual checks.
- **Research comes before choosing and planning.** The model's memory of
  libraries is often a version behind; the `researcher` reads the current
  documentation. In the reference project's twentieth phase one researcher
  looked up Samba, Apache, vsftpd, PAM, btrfs snapshots and the provider's
  snapshot API in one run: 228 turns, a context up to 200k tokens and
  24M tokens, the most of any agent of the phase. Every turn sends the
  whole context again, so the planner groups the research by the sources
  it needs and each group gets a researcher of its own, two at a time;
  the parts are joined into one `RESEARCH.md` by a shell command, so the
  main session does not read them.
- **The riskiest assumptions are tried out before anything is built.** The
  `feasibility-checker` may write only in `research/experiments/`, installs
  nothing and gets no elevated rights; what remains is the result and the
  commands to repeat it, not the code.
- **A change that adds a component is checked like the spec.** In the
  reference project `/wcc:change` took Storage Boxes into the first
  milestone: a second VM with btrfs quotas, snapshots and six file
  protocols for chrooted subaccounts, three new phases. Only the
  `researcher` ran, for the new API. Nothing tried the storage VM, and
  three requirements bundled a dozen promises each (CRUD and eight
  actions; five protocols; snapshots, plan, limits and rollback), which
  the spec-reviewer's atomicity check exists to catch. So a change that
  adds a component or phases runs the `feasibility-checker` on what it
  added and the `spec-reviewer` on the new and changed requirements.
- **Target vision apart from the first milestone.** The architecture is
  designed for the target vision, code is written only for the current
  phase.
- **User interfaces are specified up front** in UI-SPEC.md, or agents
  building views in parallel each invent their own look.
- **Documentation is asked for in the interview and written per phase** by
  the `doc-writer`, which reads the code and runs every example. Its
  requirements are in category `DOC` and hold at every close.
- **Missing credentials and rights are reasons to stop, not bugs.** The
  spec names them, `/wcc:prepare` checks them while the user is there, and
  no agent obtains elevated rights or works around a missing key. A
  credential may come from the login of a tool the user already has (a
  cloud CLI context); then it is used through that tool and not copied
  into `.env`, and `.env.example` exists only when there is a variable.
  The access agents need against a running instance of the project counts
  too: in the reference project's fifth and sixth phases the `doc-writer`
  tried to create a project and a token in the local cloud to run the
  README examples, was refused by the permission classifier, and the
  examples stayed unrun. Agents never create credentials; the spec names
  that access, optional until the phase that builds it. In the seventh
  phase the named access existed, and the permission classifier still
  refused the doc-writer's writes through it while the main session was
  allowed the same commands; `/wcc:prepare` therefore adds an allow rule
  for exactly that tool and context, and the doc-writer calls the tool
  literally so the rule matches. In the eighth phase the classifier refused
  the doc-writer's literal calls despite the rule: a project's allow rules
  apply only after the folder is trusted, and the classifier reads trusted
  infrastructure (`autoMode.environment`) only from the user's own or
  managed settings, so a repository cannot grant itself access.
  `/wcc:prepare` therefore hands the user a ready entry for their own
  settings, and the main session runs once itself what the doc-writer was
  refused: in the seventh phase it was allowed the same commands, with
  the user's conversation in view. In the ninth phase that entry let the
  doc-writer through, but the classifier refused an implementer's stop of
  the local service it had to restart, while the doc-writer had been
  allowed the same command minutes before; the entry therefore also names
  the command that starts and stops the instance. In the eighteenth phase
  the classifier refused an implementer's change that made a script read
  the token out of the cloud CLI's login for a second endpoint, as SPEC.md
  prescribed; the CLI could make that call itself with `--endpoint`, and
  did after the change. So the spec lets the tool make the call and reads
  a credential out of it only for a program that cannot use the tool. At
  the same close the main session declined to run the doc-writer's
  refused example "not past the refusal", although the skill told it to;
  run on the user's word, it was allowed. The skill now says why that is
  no way around the refusal.
- **Tools a phase uses first are checked at its start.** `/wcc:prepare`
  checks the machine once, against the stack in SPEC.md. In the reference
  project's twelfth phase, the first to build the installer, the WiX
  extensions it needed were named nowhere; a plan found them missing, the
  main session's attempt to add them was refused by the permission
  classifier, and two plans blocked until the user installed them. So the
  planner lists in "Discuss" each tool, extension or plugin the phase
  uses first that the package manager does not install, the main session
  runs its check, and the user installs what is missing while there.
- **The project folder is the only source.** In the reference project,
  `/wcc:prepare` searched sibling folders for a module path, found an older
  copy of the project and took its scaffold over, with decisions nobody had
  approved for the new one. The spec names the module or package name, and
  what is missing is asked.
- **The project name may come later.** A working title is allowed; the
  folder is renamed before `/wcc:prepare`.

### Tests

- **Test what the requirements promise, once, at the cheapest level that
  proves it.** Every requirement has a test level, confirmed by the user
  with its cost: `unit` (seconds), `e2e` (minutes), `none` (wiring a
  behaviour test covers), `manual`. Rules that only said "at least one
  test" let agents test every promise in every layer and every edge case
  they could think of; tests then outgrew the product two to one.
- **Tests are planned before they are written.** Each phase writes a test
  map: one row per promise, its level, its test name. A promise is tested
  where it arises, behaviour many resources share is tested with all its
  cases where it is implemented and with one case per resource to show it
  is wired in, glue code and configuration get no test, and only edge cases the
  requirement names, that happen in normal operation, or that would destroy
  data or leak a secret. The map is checked before the tests are written;
  afterwards a Grep finds every test name, and finding none is a failure.
  The rule for shared behaviour once said only "one table-driven row over
  all of them"; in the reference project's second phase the planner tested
  paging, filters, sorting and labels where they were implemented and then
  again with every sort key and filter on every endpoint, and test code
  grew to 1.5 times the product code.
- **Tests come first, from a separate agent.** The `test-writer` does not
  see the implementation and writes exactly the rows. Tests written by the
  agent that implements would check what it built, not what was asked; a
  separate writer keeps an objective "done" for a phase that runs without
  questions.
- **A writer's share is limited like a plan.** In the reference project's
  thirteenth phase, the client matrix, two writers wrote about 3,000 lines
  of end-to-end tests each; their context grew to 232k tokens and they used
  36M tokens, nearly as much as all nineteen implementers. Plans had a size
  limit, writers had none. So the planner splits a writer whose rows come
  to more than about 800 lines of tests into parts by test file, run one
  after another, the part with the shared harness first.
- **Only what a change can break runs during development.** A plan runs its
  fast checks while it repairs and its own end-to-end tests once they are
  green. A phase's close runs the unit suite and the linter in full and the
  end-to-end tests of what it covers and changed, by test function. All
  end-to-end tests run at the end of a milestone, before its tag. A green
  test is not run again on unchanged code.
- **End-to-end tests share what is expensive**: one environment per run,
  one run of an expensive step for all checks that need it, the
  environment's lifecycle as one sequence, booted machines shared, waiting
  subtests in parallel, time limits tested with a test clock.
- **Parallel tests take a shared limit in one step, and a run shows its
  progress.** In the reference project's fifth phase the close's full
  end-to-end run hung for over an hour: rows that need two servers took
  their slots one after another, and three of them each held one slot
  while waiting for another. The run had a limit of three hours and logged
  nothing until the end, so the hang showed only as time passing; after
  the fix it took 22 minutes.
- **Every waiting step has its own deadline.** In the reference project's
  eleventh phase a browser test read a form control before the form had
  rendered; the framework's click then waited for an element that never
  came, without a limit of its own, until the test's limit of 40 minutes,
  and printed nothing meanwhile. The rule against fixed waits was there,
  but the test-writer did not take the framework's built-in waits for
  waits. With a deadline per action the next run named the control after
  76 seconds.
- **Findings about the tests get no tests about the tests.** The proof is
  that existing tests stay green, plus a measurement.

### Implementation

- **One phase per run, started by the user.** Every phase reads the project
  files afresh, so what an earlier one left in the context helps nothing
  and is sent again with every call.
- **The main session stays small.** Everything in its context is sent
  again with each of its turns, and a phase has about two hundred. In the
  reference project's first phase under 0.31 the main session read and
  wrote the whole planning before the first plan started and carried that
  through every later turn: its context grew from 80k to 430k tokens and it
  used more than all implementers together. So the `planner` agent reads
  the planning documents and the code, drafts the assumptions of
  "Discuss" and writes PLAN.md and TESTS.md; the main session gets the
  waves. It reads STATUS.md and ROADMAP.md by the parts it needs; output
  goes to files; agents get the plan ID and the phase folder instead of
  copied text, read what the plan's `Reads:` line names, and return a short
  result in a fixed form; the `researcher` writes RESEARCH.md itself. STATUS.md
  lines stay one or two sentences and a finished phase shrinks to one line
  in ROADMAP.md, its details moving to ROADMAP-DONE.md, so the files every
  skill reads stay small.
- **Plans are small**: one component, one to three tasks, no task over five
  files, about 400 lines, also when a plan bundles backlog items. An agent
  sends its whole context with every turn, so a large plan costs more than
  two halves; in the reference project one oversized plan used a fifth of
  its phase's tokens.
- **Waves are computed when planning**, from the dependencies and the files
  of each plan. A plan that depends on another starts only after
  that one is reviewed.
- **Two agents at the same time, fixed.** In the
  reference project two agents finished a phase in 1:25 h instead of 2:09 h
  with fewer tokens and no conflicts; each agent costs the same alone or in
  parallel. An option for it was dropped.
- **Plans carry what agents share.** "Interfaces" in PLAN.md holds the
  signatures new in the phase that tests and several plans build on, so
  test-writers working per component before any code exists agree with
  each other and with the implementers.
- **Implementation runs in its own agent**, the `implementer`, with only the
  tools it needs and its fixed rules, so the prompt holds only the plan ID.
- **Agents wait for their own commands inside their turn**, with a
  foreground loop for long ones; a command still running when an agent
  returns wakes the main session again and again.
- **No debug access in the product, no way around a protection.** The
  rules forbid it and the reviewer searches every diff for it.
- **Repairing and adding is allowed, changing and replacing stops the
  plan.** When in doubt, it stops. The rule protects what others build on;
  what a plan's own change leaves without any caller may go, also from an
  internal interface, while a public one (API, file format, command line,
  configuration) keeps it. In the reference project's first backlog phase
  a plan replaced the per-server metrics call with one for all servers,
  the old method stayed in the platform interface with no caller, and the
  phase asked the user whether to remove it.
- **Three repair attempts, then block late.** Dependent plans are skipped,
  the rest is finished, then `BLOCKED` follows.
- **A failure from outside the plan goes to a fresh implementer.** In the
  reference project's third phase the rebuild plan was written in minutes;
  its end-to-end run then hit three defects of other components, and the
  same agent chased them for two hours with a context of up to 250k
  tokens, more than half of all implementer tokens of the phase.
  Experiments did not count as repair attempts. Now the implementer stops
  once its own part passes the fast checks, and a new one fixes the cause
  from a small context; one review covers the plan and the fix.
- **Only the main session writes the shared documents**; subagents report.
  The exceptions are the phase's PLAN.md and TESTS.md, which the `planner`
  writes first, and its RESEARCH.md, which the `researcher` writes.
- **Git rules for parallel agents**: stage by name, no stash, no reset, one
  branch per plan; worktrees are removed after their merge without force.
  A worktree git calls modified with an empty `git diff` has a stale index
  entry (in the reference project the size of a file's CRLF version after
  it was rewritten with LF); `git add` of the named files clears it, where
  `--force` would also discard real changes.
- **One branch per phase**, so the base branch holds only finished phases;
  one tag per milestone. Skills that change planning documents between
  phases commit them, and a phase commits what another skill left in
  `.project/` on the base branch before it branches: in the reference
  project `/wcc:change` settled an open decision without committing it,
  and the next phase took the changed files along on its own branch, so
  the base branch lacked the decision until that phase was merged.
- **No falling back to a similar package name**: a near miss may be
  malicious.

### Reviews and backlog

- **Reviews have a stopping rule**: blockers are fixed, one round per plan,
  the architecture is not up for debate.
- **The reviewer removes what cannot be needed**: a guard for a state never reached, a check made elsewhere, an
  option nothing sets, a generalisation with one use. Only with evidence
  from the repository. It is the counterweight to agents that harden
  everything.
- **The backlog is a parking lot.** An item needs a severity (`security`,
  `data`, `bug`, `later`, `minor`) and a stated consequence; at most two per
  review and verifier, usually none. Nothing in it is scheduled by itself:
  `/wcc:phase` and `/wcc:status` show the count per severity, and the user
  takes items in with `/wcc:change`, which makes them a backlog phase. With
  automatic backlog milestones, reviewers filled their quota and those
  milestones took longer than feature work while writing mostly tests.
- **The phase is checked as a whole.** The `verifier` works backwards from
  the success criteria and requirements, checks every row of the test map,
  the documentation and ARCHITECTURE.md against the code, and believes no
  report.
- **ARCHITECTURE.md holds what other components build on**; how a component
  works inside goes into code comments.

### Manual checks and feedback

- **Manual checks have a guide** in `CHECKS.md`, written at the close while
  the code is fresh, and stop a phase only if later phases depend on them.
- **`/wcc:report` walks the user through them** one at a time: Claude
  prepares what it can, says what should happen, and
  saves each answer at once.
- **Findings are sorted with the user**: bugs become a feedback phase with a
  failing test first, minor findings are parked, and a wish is worked into
  the documents by the steps of `/wcc:change` and then built like a bug.
  Whatever concerns a phase waiting in `MANUAL_CHECK` goes into that phase.
  While a phase is running they wait in FEEDBACK.md.
- **Report is for what is built, change for what is new.** The line ran
  first between recording (`/wcc:report`) and changing the documents
  (`/wcc:change`); in the reference project's fourteenth phase the user's
  findings were mostly wishes about the console just built, each went from
  report to change, the user wanted all of them in the waiting phase, which
  no skill offered, and found the difference between the two skills hard to
  explain to others. Users think in "I noticed" and "I want something new",
  so the skills split that way.

### State

- **State lives in files, not in the context**, so phases can be
  interrupted and resumed: check marks in the plan, phases as roadmap lines.
- **DECISIONS.md holds only what is valid.** Agents find entries by
  searching for a component, so a revoked entry next to the one that
  replaced it shows two answers, and only the dates tell them apart. In
  the reference project the file had grown to 343 entries by the tenth
  phase; no agent had yet taken a revoked entry for valid, the rule
  came before that. An entry that revokes or changes one names it under
  `Replaces:`, and the old one moves to DECISIONS-DONE.md, as finished
  phases and backlog items do.
- **`/wcc:upgrade` brings projects along.** It knows no project, only the
  migration catalogue and the current templates, changes the form of the
  files, never their decisions, and asks once.

## Bringing existing projects along

Whoever changes something that affects existing projects adds an entry to
`skills/upgrade/references/migrations.md` in the same change: the version,
how to detect that it is missing, what `/wcc:upgrade` changes, and which
skill runs afterwards. A project is affected when a skill or agent newly
expects, renames or reads in another form a section, line, term or path in
the project files. Rules only in `project-CLAUDE.md` need no entry. The
detection checks file content, never project names or history.

## Customising

| Wish | Where |
|------|-------|
| Fixed conversation language | Best in your own `~/.claude/CLAUDE.md`. In the plugin: the paragraph starting "Talk to the user in the language they have configured" in every `SKILL.md`, and "Language and documents" in `skills/idea/assets/project-CLAUDE.md` |
| Different document language | Language rule in every `SKILL.md` and in `project-CLAUDE.md`, templates in `assets/`, return language in `agents/*.md`, and every key term (see "Language") |
| Rules during a phase (questions, what is allowed, stop conditions) | "During a phase" in `skills/idea/assets/project-CLAUDE.md`; "Plans that cannot be finished" in `skills/phase/SKILL.md` |
| What the start of a phase asks | "Discuss" in `skills/phase/SKILL.md`, step 1 of `agents/planner.md` |
| Interview questions and mandatory sections | `skills/idea/SKILL.md`, `skills/idea/assets/IDEA.md` |
| Another project type | One section each in `skills/idea/references/project-types.md` and `skills/spec/references/project-types.md` |
| Form of requirements and test levels | Step 2 of `skills/spec/SKILL.md`, `skills/spec/assets/REQUIREMENTS.md`, `agents/spec-reviewer.md` |
| Phases and success criteria | Step 4 of `skills/spec/SKILL.md`, `skills/spec/assets/ROADMAP.md` |
| Plans, tasks, waves, size limits | Step 2 of `agents/planner.md` |
| Parallelism, order of the run | Steps 7 to 9 of `skills/phase/SKILL.md` |
| How tests are planned and how many | Step 2 of `agents/planner.md` (test map), `agents/test-writer.md`, "Tests" in `project-CLAUDE.md` |
| End-to-end rules, what runs when | `skills/phase/references/e2e.md`, "Tests" in `project-CLAUDE.md` |
| Stricter or looser reviews | `agents/code-reviewer.md`, `agents/verifier.md`, `agents/spec-reviewer.md` |
| What enters the backlog, how it is worked off | `skills/phase/references/backlog.md`, the reviewer and verifier, "Backlog" in `skills/change/SKILL.md`, template `BACKLOG.md` |
| Manual checks and findings | `skills/report/SKILL.md`, `skills/phase/references/feedback.md`, "Manual requirements" in `skills/phase/SKILL.md` |
| Open decisions | "Open decisions" in `skills/phase/SKILL.md`, the agents' categories, "Open decisions", "Asking" and "Clarification sessions" in `skills/change/SKILL.md`, template `STATUS.md` |
| Clarification sessions | Step 2 in `skills/idea/SKILL.md` and "Clarification sessions" in `skills/change/SKILL.md`; change both |
| How plans are implemented | `agents/implementer.md`; its prompt in step 8 of `skills/phase/SKILL.md` |
| What is read and how | "Reading" in `skills/phase/SKILL.md` and `agents/planner.md`, the opening paragraphs of the other agents |
| Folder for the planning documents | "All planning documents live in …" in every `SKILL.md`, `project-CLAUDE.md`, the agents, README |
| Documentation of projects | Interview question in `skills/idea/SKILL.md`, `project-types.md`, "Documentation" in the IDEA and SPEC templates, `DOC` in `REQUIREMENTS.md`, `agents/doc-writer.md`, `agents/verifier.md` |
| Approval after the spec, stack criteria | Steps 7 and 1 in `skills/spec/SKILL.md` |
| Machine checks | Step 1 in `skills/prepare/SKILL.md` |
| Branch and tag names | `skills/phase/SKILL.md` |
| Model of an agent | `model` in the header of `agents/*.md` |

A rule that appears in several places is changed in all of them; the
project CLAUDE.md and the agents must agree.

## Testing changes

```
claude --plugin-dir .
```

Check manifest and marketplace with `claude plugin validate .`, then try
the skills in an empty test project; reload the plugins after editing a
skill or agent. Two test cases cover most of it: a small project with fixed
rules (a Tetris-like game) and a project that starts small with a large
target vision (a re-implementation of an existing API with many resource
types).

## Conventions when editing

- The `description` of a skill decides when it triggers; keep the English
  trigger phrases. They match input in other languages by meaning.
- State a rule once, with its reason, in the place that uses it. Do not
  repeat it with emphasis, and do not add an example of an incident to a
  skill or agent: agents read every sentence as an instruction, and a
  growing list of special rules makes them do more, not better. Reasons and
  history belong in this file.
- Keep every `SKILL.md` short; what applies only sometimes goes into
  `references/` and is read when it applies.
- Agents that only judge have read access (`tools: Read, Grep, Glob`); the
  `code-reviewer` also has `Bash` to read diffs and run tests and must not
  change files. The `feasibility-checker` writes only in
  `research/experiments/`. The `implementer` has no web access; what it
  needs to know comes from the `researcher` through the phase folder.
- Raise `version` in `plugin.json` with every change of behaviour, with a
  migration entry if projects need it.
- Adjust author and owner in `plugin.json` and `marketplace.json` when
  passing the plugin on under your own name.

## Decisions taken

- **The approval after the spec stays**: the one fixed stop before
  anything is built.
- **Milestone, phase, plan, task, wave, requirement, success criteria** as
  the words for the levels of work, each with one meaning (see "Words").
- **Planning documents are English**, so others can work on a project; the
  conversation follows the user.
- **`.project/` as a hidden folder**, with a folder per phase.
- **`/wcc:prepare` is a skill of its own**: machine check and scaffold in
  one place, repeatable on new machines and checkouts.
- **`.env` is protected by a deny rule**, not a hook, because a hook needs
  a script for every operating system.
- **No options.** Parallelism is fixed at two, end-to-end tests run in full
  at the end of a milestone, the backlog runs when the user takes items in.
  Each former option had one sensible value in practice. A model profile
  per project was considered and left out too; the agents keep their fixed
  `model:`.
- **Cheaper model** (assumption): `spec-reviewer` and `researcher` run with
  `model: sonnet`; all others inherit the session's model.
- **No migration from versions before this one.** The vocabulary, the
  folder and the documents changed completely; the only project, the API
  test project, starts again from scratch. The migration catalogue starts
  empty, and `/wcc:upgrade` handles changes from here on.
- **No goal that chains phases** (such as `/goal`): it would bring back long
  sessions over several phases.
- **Postponed**: a short path for single fixes after `DONE`, and support for
  projects not started with wcc.
- **Optional prototype** (assumption): `/wcc:prototype`, recommended in the
  interview for games whose rules are unclear.
- **The skill for manual checks and findings is `/wcc:report`**, not
  `/wcc:feedback`: the slash menu of Claude Code Desktop hides a plugin
  skill named exactly `feedback` (probes named `feedback-probe` and one
  with the same description were shown), although it runs when typed.
  The files and the feedback phases keep their names.

## Open points

1. **Nothing of this version has run yet.** The plan is to build the API
   test project again from scratch with it and compare against the previous
   run: test code against product code, minutes per phase, backlog items
   per phase, tokens per agent.
2. **Session tools.** `/wcc:idea` and `/wcc:change` use
   `mcp__ccd_session__start_session` (Claude Code Desktop). Whether the
   child session starts at once or shows a card first is untested; the
   skills cover both.
3. **Not tried in tools:** whether `TaskStop` works in the `tools:` of a
   plugin agent. `PowerShell` does.
4. **Two ideas not built:** a script that lists tests without coverage of
   their own, and mutation testing as a one-off quality check.
