# wcc

A Claude Code plugin that takes a project from a raw idea to working,
tested code, with as little of your time as possible once the idea is clear.

You describe what you want. Claude interviews you until the idea is
complete, picks and checks the technology, writes the requirements, a spec
and a roadmap, and then builds the project phase by phase: tests first,
reviewed, without asking you anything while it works. You start each phase
with one command and can leave it alone until it is finished.

## How it works

```
  you                         Claude
  ───                         ──────
  /wcc:idea  "I want …"  ──►  interview, a few questions at a time  ──►  IDEA.md
  /wcc:spec              ──►  stack, feasibility, requirements, plan ──►  summary for your OK
  /wcc:prepare           ──►  checks your machine, sets up the repo  ──►  build and tests run
  /wcc:phase             ──►  asks what is unclear, then plans,      ──►  one phase per call
                              tests, builds, reviews on its own
  try it, /wcc:report    ──►  walks you through the checks           ──►  fixes in the next phase
```

Your attention goes into the interview, the approval of the spec and the
start of each phase. Questions are cheap while you are at the keyboard and
expensive while a phase runs, so wcc asks up front and decides the rest on
its own, writing down every decision so you can review it.

## Words

| Word | Meaning |
|------|---------|
| **Milestone** | A version of your project, for example the first version. It ends with all end-to-end tests and a git tag. |
| **Phase** | A slice of a milestone that runs and can be checked; `/wcc:phase` builds one. Numbered `01`, `02`, …; `F1` fixes what you reported, `B1` works off backlog items you chose. |
| **Plan** | One piece of work inside a phase, for one component, done by one agent (`03-02`). |
| **Task** | A step inside a plan, one to three per plan. |
| **Wave** | Plans that do not depend on each other and may run at the same time. |
| **Requirement** | One checkable capability, such as `AUTH-03 user can reset the password with a link from an email`, with its test level. |
| **Success criteria** | Two to five results of a phase you can observe, such as "a new user can sign up and log in". |

Three ideas carry the workflow:

- **The idea is yours, the how is Claude's.** The interview asks only what
  you can answer: the problem, goals, non-goals, how big it may grow, what
  success looks like. Every question comes with options and a
  recommendation. Technical choices are researched against current
  documentation and, where risky, tried out before anything is built.
- **Done means tested, and no more than that.** Every requirement has a test
  level. Each phase first plans its tests in a test map, one row per promise
  at the cheapest level that proves it; then a separate agent writes exactly
  those tests before the code exists, so they check what you asked for, not
  what the code happens to do.
- **Built small, designed for where it is going.** The architecture is
  designed for your target vision; code is written only for the current
  phase. Later milestones build on the right interfaces instead of a
  rewrite.

## Requirements

- [Claude Code](https://claude.com/claude-code). The desktop app is
  recommended: questions that need a longer discussion, such as the choice
  of stack, can move to a side session there.
- git, and whatever tools your project's stack needs. `/wcc:prepare` tells
  you what is missing; it does not install system software itself.

## Install

From a git repository that hosts the plugin:

```
/plugin marketplace add <owner>/<repo>
/plugin install wcc@webcodingcat
```

Or try it from a local copy:

```
claude --plugin-dir /path/to/wcc
```

## Quick start

Open Claude Code in an empty folder and describe your idea:

```
/wcc:idea I want a small tool that backs up my photos to two cloud services
```

Answer the questions, continue with `/wcc:spec`, approve the summary, run
`/wcc:prepare`, and start building with `/wcc:phase`. `/wcc:status` always
tells you where the project stands and which command comes next.

## The steps in detail

### 1. Idea: `/wcc:idea <your idea>`

An interview in rounds of at most three questions. Each offers a few
options, marks one as recommended and lets you say "I don't know" or ask
for more details in a separate session. Claude lists the assumptions it
would make for you to confirm. A separate agent then tries to sketch an
architecture from your answers alone and reports what it had to guess. The
result is `IDEA.md`: what and why, never how. It is your document.

The interview also asks who needs documentation and, at the end, for a
name; a working title is fine. Rename the folder between `/wcc:idea` and
`/wcc:spec` if you like: nothing depends on the path until `/wcc:prepare`.

**Not sure what you want?** `/wcc:prototype <what to try>` builds a quick
throwaway prototype first, for example to find out how a game should feel.

### 2. Spec: `/wcc:spec`

Claude chooses the stack with current versions and writes:

- `REQUIREMENTS.md`: one line per requirement, grouped by category, each with
  its test level:
  - `unit`: checked in code, in seconds
  - `e2e`: checked against the real environment or a browser, in minutes
  - `none`: wiring or configuration that the tests around it cover
  - `manual`: by you, because no test can judge it
- `SPEC.md`: stack, references the project must match, accounts and
  secrets (by name), machine prerequisites, documentation
- `ARCHITECTURE.md`, and `UI-SPEC.md` for projects with a user interface
- `ROADMAP.md`: the first milestone in phases, each with its requirements
  and success criteria; the rest of your target vision under "Later"

Risky technical assumptions are checked first, by research or a small
experiment; if that needs a program you do not have or administrator
rights, Claude asks you. Decisions that are expensive to change later (data
schema, file formats, public interfaces, authentication) are settled here.
You get a short summary with what the test levels cost and approve it. This
approval is the one fixed stop: nothing is built before it.

### 3. Prepare: `/wcc:prepare`

Checks that your machine has what the project needs, including accounts and
secrets (by name; the values stay in your `.env`, which Claude may not
read), creates the git repository and the scaffold, installs the
dependencies and proves that build and tests run. Run it again on a fresh
checkout or a new machine.

### 4. Phases: `/wcc:phase`

Each phase runs the same loop:

1. **Discuss.** While you are there, Claude reads the requirements and the
   code and shows how it would build what they leave open, as a short list
   you confirm or correct. While you read it, anything that depends on
   how a library, tool or outside service behaves is looked up; only the
   open questions that remain are asked, with a recommendation each.
2. **Plan** the phase in plans with one to three tasks each, sorted into
   waves, after looking up anything used for the first time.
3. **Plan the tests** in a test map, one row per promise of its
   requirements, and **write exactly those**, before any code.
4. **Implement** the plans wave by wave, up to two agents at the same time.
5. **Review** each plan once. Only blockers are fixed: a bug, a violation of
   a requirement, a security problem, tests that do not match the map, or
   code that provably cannot be needed. What would change the requirements
   or the architecture is left for you as an open decision.
6. **Document** what was built, if the spec asks for documentation.
7. **Check** the phase as a whole against its success criteria: tests green,
   nothing stubbed, everything wired in, the documentation true.

Each phase is built on its own branch and merged into yours only when it is
finished. During development only the tests a change can break run; all
end-to-end tests run at the end of a milestone, before its tag
`wcc-milestone-<n>`.

Then the phase stops and names the next command. **Each phase is started by
hand**, best in a new session or after `/clear`: every phase reads the
project files afresh, so the old conversation would only fill the context.

**During a phase Claude asks nothing after the start.** When something is
unclear, it picks the simplest solution that fits the requirements and
records it in `DECISIONS.md`. It may repair and add (fix a bug, add input
validation, extend an interface without breaking it), but not change or
replace (rename an interface, swap a library, drop a component). Those, and
consequential decisions nothing settles, stop the plan. A stuck plan does
not stop the phase: everything that does not depend on it is finished
first.

**What only you can decide waits under "Open decisions"** in
`.project/STATUS.md`, each as one question with options and a
recommendation. The next phase asks those that concern it at its start;
`/wcc:change` takes answers any time.

**The backlog is a parking lot** for findings with a real consequence, each
with a severity (`security`, `data`, `bug`, `later`, `minor`). Nothing in it
runs by itself: every phase and `/wcc:status` show how many items wait, and
`/wcc:change` lets you take items in, keep them or drop them. Items you take
in become a backlog phase.

## Leaving a phase alone

Switch the session to auto mode after the start of `/wcc:phase`, so
permission prompts do not stop it. A phase ends with one of these in
`.project/STATUS.md`:

| Status | Meaning | What you do |
|--------|---------|-------------|
| `IN_PROGRESS` | The phase is finished, more are open in this milestone | Start the next one with `/wcc:phase`, best in a new session |
| `DONE` | The milestone is finished | Try it; plan the next milestone with `/wcc:change` |
| `MANUAL_CHECK` | A later phase depends on something only you can check | `/wcc:report` walks you through it; then `/wcc:phase` continues |
| `BLOCKED` | A plan needs a decision, an approval, a credential or a step only you can take | `/wcc:phase` shows the question and continues after your answer |

## Trying it and reporting what you find

`/wcc:report` without text walks you through the open manual checks one
at a time. Claude prepares what it can (starts the program, creates the
test data, opens the page), tells you what should happen and asks whether it
does. Answer "fits", describe what is different, or say "skip" or "not
now"; each result is saved right away.

Anything else you notice goes the same way, in your own words and with a
screenshot if it helps:

```
/wcc:report the score does not reset after game over
```

Claude looks into it, asks if something is unclear, and shows you how it
sorted each finding before recording it:

- a **bug** becomes part of a feedback phase `F<n>`, built like any other:
  a failing test first, then the fix
- a **small thing** is parked in the backlog; if you want it fixed now, say
  so and it goes into the feedback phase instead
- a **wish**, something that should work or look differently than the
  requirements say, is worked into the documents right away and then built
  like a bug

Anything about a phase that waits for your manual check goes into that
phase, whatever it is, so it is finished before the phase is merged.

You can report findings while a phase is running; they wait in
`.project/FEEDBACK.md` until it has finished.

## Changing plans and growing the project

`/wcc:report` is for what you notice in what is built; `/wcc:change` is for
new ideas and for how the project goes on.

`/wcc:change <what you want>` works a change into all documents
consistently: a new requirement, a dropped one, a different library, a
bigger target vision. When a milestone is `DONE`, it plans the next one from
the target vision. Without text, it shows the open decisions and how many
backlog items wait, and takes your answers; "the backlog" lists the items
so you can take them in, keep or drop them.

## After a plugin update

The rules in your project's `CLAUDE.md` are a copy made when the project
started. Run `/wcc:upgrade` in each project after updating the plugin;
`/wcc:status` and `/wcc:phase` tell you when a project is behind. It shows
what it would change and asks once; `/wcc:upgrade --check` only shows it.

## Commands

| Command | When | What it does |
|---------|------|--------------|
| `/wcc:idea <idea>` | start | Interview, then `IDEA.md` |
| `/wcc:spec` | after the interview | Stack, feasibility check, requirements, spec, architecture, roadmap, summary for approval |
| `/wcc:prepare` | after the approval, on a new machine or checkout | Machine, accounts and secrets checked, git repository, scaffold, working build and test commands |
| `/wcc:phase` | to build | The next phase: discuss, plan, test, build, then it stops; start each one in a new session |
| `/wcc:report [what you noticed]` | any time | Without text: the open manual checks, one at a time. With text: a bug or a wish about what is built, sorted with you and recorded where it is built |
| `/wcc:change [change]` | any time | A new idea worked into all documents, the next milestone planned, open decisions answered, backlog items taken in |
| `/wcc:status` | any time | Where the project stands and which command comes next |
| `/wcc:upgrade [--check]` | after a plugin update | Project files brought up to the installed plugin version |
| `/wcc:prototype <what to try>` | optional, before the interview | Throwaway prototype to find out what you want |

## What wcc puts in your project

- `CLAUDE.md` in the project root: the rules every Claude session in the
  project follows, and the build and test commands.
- `.project/`, a hidden folder with the planning documents. `IDEA.md` is
  yours; `REQUIREMENTS.md`, `SPEC.md`, `ARCHITECTURE.md`, `ROADMAP.md` and
  `UI-SPEC.md` describe the plan; `STATUS.md`, `DECISIONS.md` and
  `BACKLOG.md` track the state, the open decisions, every decision taken
  without you, and the parked findings. `ROADMAP-DONE.md` and
  `BACKLOG-DONE.md` keep the details of finished work, `DECISIONS-DONE.md`
  the decisions that were revoked or changed later. `research/` holds what
  the feasibility check found, `decisions/` the results of clarification
  sessions, and `phases/<id>/` everything of one phase: what you said at its
  start, its plans, its test map, its research and its manual checks. You
  only need `IDEA.md`, `STATUS.md` and, now and then, `DECISIONS.md`.

The planning documents are not your project's documentation. If you asked
for user or developer documentation, the spec lists those documents with
their paths outside `.project/`, and every phase brings them up to date.

## Language

The plugin does not set the language of the conversation: Claude talks to
you in the language you write in or have configured. Everything it writes to
the repository is English, the planning documents included, so other people
can work on the project.

## Status of the plugin

Work in progress. `CLAUDE.md` in this repository is for developing the
plugin: it explains the design behind the workflow, where to change what,
and the open points.
