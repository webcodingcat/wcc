# wcc

A Claude Code plugin that takes a project from a raw idea to working,
tested code, with as little of your time as possible once the idea is clear.

You describe what you want. Claude interviews you until the idea is
complete, picks and checks the technology, writes a spec and a plan, and
then builds the project milestone by milestone: tests first, reviewed,
without asking you anything while it works. You can leave it running
overnight and come back to finished milestones.

## How it works

```
  you                         Claude
  ───                         ──────
  /wcc:idea  "I want …"  ──►  interview, a few questions at a time  ──►  IDEA.md
  /wcc:spec              ──►  stack, feasibility check, spec, plan   ──►  summary for your OK
  /wcc:prepare           ──►  checks your machine, sets up the repo  ──►  build and tests run
  /wcc:milestone         ──►  plans, writes tests, builds, reviews   ──►  milestone after milestone
                               … unattended …
  try it, /wcc:feedback  ──►  sorts what you found into the plan     ──►  fixed in the next run
```

Your attention goes almost entirely into the first two steps. That is
deliberate: questions are cheap while you are at the keyboard and expensive
while a run is going, so wcc asks everything up front and decides the rest
on its own later, writing down every decision so you can review it.

Three ideas carry the workflow:

- **The idea is yours, the how is Claude's.** The interview asks only what
  you can answer: the problem, goals, non-goals, how big it may grow, what
  success looks like. Every question comes with options and a
  recommendation. Technical choices are made by Claude, researched against
  current documentation and, where risky, tried out in a small experiment
  before anything is built.
- **Done means tested.** Every requirement has acceptance criteria. A
  separate agent writes the tests from the spec before the code exists, so
  the tests check what you asked for, not what the code happens to do. A
  milestone is finished when all tests pass, a reviewer found no blocker
  and a final check found nothing that is only stubbed or not wired in.
- **Built small, designed for where it is going.** You describe a target
  vision, the version you have in mind eventually, and a first version. The
  architecture is designed for the target vision; code is written only for
  the current milestone. Later stages build on the right interfaces instead
  of a rewrite.

## Requirements

- [Claude Code](https://claude.com/claude-code). The desktop app is
  recommended: during the interview, questions that need a longer
  discussion, such as the choice of stack, can move to a side session there.
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

Answer the questions. When the interview is done, continue with
`/wcc:spec`, approve the summary, run `/wcc:prepare`, and start the
implementation with `/wcc:milestone`. If you lose track at any point,
`/wcc:status` tells you where the project stands and which command comes
next.

## The steps in detail

### 1. Idea: `/wcc:idea <your idea>`

An interview in rounds of at most three questions. Each offers a few
options, marks one as recommended and always lets you say "I don't know"
or ask for more details. Claude also lists the assumptions it would make
for you to confirm, which is faster than answering twenty more questions.

The interview is finished when every mandatory section is filled and a
separate agent, trying to sketch an architecture from your answers alone,
finds nothing it has to guess. The result is `IDEA.md`: what and why, never
how. It is your document; read it, and change it when your idea changes.

The interview also asks who needs documentation (end users, developers,
both or nobody) and, at the end, for a name. A working title is fine.

**No name yet?** Start in a folder with a codename, for example
`ideas/falcon`. Rename the folder between `/wcc:idea` and `/wcc:spec`:
close the session, rename it, open a new session there. Nothing depends on
the path until `/wcc:prepare`, which asks once more if the name is still
open.

**Not sure what you want?** `/wcc:prototype <what to try>` builds a quick
throwaway prototype first, for example to find out how a game should feel.
What you learn from it feeds the interview; the code is thrown away.

### 2. Spec: `/wcc:spec`

Claude chooses the stack with current versions, writes the requirements
with acceptance criteria, the architecture and a roadmap of milestones.
Risky technical assumptions are checked first, by research or by a small
experiment; if that needs a program you do not have or administrator
rights, Claude asks you to install it or run the command.

Decisions that are expensive to change later (data schema, file formats,
public interfaces, authentication) are settled here and shown to you. You
get a short summary and approve it. This approval is the one fixed stop in
the workflow: nothing is built before it.

Every acceptance criterion says how it is verified:

- `auto`: by an automated test
- `browser`: by a test that drives a browser
- `manual`: by you, because no test can judge it (does the animation feel
  right?)

The more `manual` criteria, the more often a run needs you. For an API
there are few, for a game many.

### 3. Prepare: `/wcc:prepare`

Checks that your machine has what the project needs, including accounts
and secrets (by name; the values stay in your `.env`, which Claude is not
allowed to read). Then it creates the git repository and the project
scaffold, installs the dependencies and proves that build and tests run.

Run it again on a fresh checkout or a new machine.

### 4. Implementation: `/wcc:milestone`

Each milestone runs the same loop:

1. **Plan** the milestone in tasks, after looking up anything used for the
   first time.
2. **Write the tests** for all its acceptance criteria, before any code,
   and commit them.
3. **Implement** the tasks, one after another or several in parallel.
4. **Review** each task once. Only blockers are fixed (a bug, a violation
   of the spec, a security problem, a failing test). What would change the
   spec or the architecture is left for you as an open decision;
   everything else goes to the backlog.
5. **Document** what was built, if the spec asks for documentation.
6. **Check** the milestone as a whole: all tests green, nothing stubbed,
   everything wired in, the documentation true.

Each milestone is built on its own branch `wcc/m<n>` and merged into your
branch only when it is finished, so a run that stops never leaves half a
milestone there. A finished stage is tagged `wcc-stage-<n>`.

Then the next milestone starts, until the stage is done or the run needs
you. After every few milestones a **backlog milestone** works off the
collected review findings.

**During a run Claude asks nothing.** When something is unclear, it picks
the simplest solution that fits the spec and records the decision in
`DECISIONS.md`, with how expensive it would be to change later. It may
repair and add things (fix a bug, add input validation, extend an
interface without breaking it), but not change or replace them (rename an
interface, swap a library, drop a component). Those, and consequential
decisions the spec did not settle, stop the task. A stuck task does not
stop the whole run: everything that does not depend on it is finished
first.

**What only you can decide waits under "Open decisions"** in
`docs/wcc/STATUS.md`: findings that would change the spec or the
architecture, each as one question with options and a recommendation. The
run goes on with everything that does not depend on them. Every report
names the new ones, `/wcc:milestone` asks at the start whether you want to
decide them first, and `/wcc:change` without text takes your answers. If a
question needs a longer discussion, answer "I need more details": as in the
interview, it is discussed in a separate session, and the next
`/wcc:change` works in the result.

## Leaving a run alone

Switch the session to auto mode before `/wcc:milestone`, so permission
prompts do not stop the run. A run ends when `docs/wcc/STATUS.md` says one
of these:

| Status | Meaning | What you do |
|--------|---------|-------------|
| `DONE` | The current stage is finished | Try it; plan the next stage with `/wcc:change` |
| `MANUAL_CHECK` | A later milestone depends on something only you can check | `/wcc:feedback` shows how to test it; `/wcc:milestone` asks for the result and continues |
| `BLOCKED` | A task needs a decision, an approval, a credential or a step only you can take | `/wcc:milestone` shows the question and continues after your answer |

If a run stops early in practice, set a goal first, for example
`/goal docs/wcc/STATUS.md says DONE, BLOCKED or MANUAL_CHECK`.

Several agents in parallel finish sooner but use up your usage limits just
as much faster, and a run that hits the limit stands still. That is why the
default is one agent; raise it with `/wcc:options`.

## Trying it and reporting what you find

`/wcc:feedback` without text lists every open manual check with a guide:
how to start the project, what to set up, which steps to take and what you
should see. Answer right there, for example "1 and 3 fit, at 2 the
animation flickers".

Anything else you notice while trying the project goes the same way, in
your own words and with a screenshot if it helps:

```
/wcc:feedback the score does not reset after game over
```

Claude looks into it, asks if something is unclear, and shows you how it
sorted each finding before recording it:

- a **bug** becomes part of a feedback milestone `F<n>`, which the next
  `/wcc:milestone` builds like any other: a failing test first, then the
  fix
- a **small thing** goes to the backlog; if you do not want to wait for
  the next backlog milestone, say so and it goes into the feedback
  milestone instead
- a **wish** beyond the spec goes through `/wcc:change`, right away if you
  like; otherwise it waits under "Open decisions"

You can report findings while a run is going. They wait in
`docs/wcc/FEEDBACK.md` until the run has finished its current milestone.

## Changing plans and growing the project

`/wcc:change <what you want>` works a change into all documents
consistently: a new feature, a dropped one, a different library, a bigger
target vision. When a stage is `DONE`, `/wcc:change` also plans the next
stage from the target vision. Without text, `/wcc:change` shows the open
decisions with a recommendation each and takes your answers. Change the idea through this command rather
than by editing one document, so spec, architecture and roadmap stay in
step with it.

## Options

`/wcc:options` sets how runs behave. The values are stored in the section
"Options" of `docs/wcc/STATUS.md` and stay in effect for later runs until
you change them:

| Option | Default | Effect |
|--------|---------|--------|
| `parallel=<1-5>` | 1 | Agents writing tests or implementing tasks at the same time. 1 runs everything one after another; more finish sooner but use up your usage limits just as much faster. |
| `once=<yes\|no>` | no | `yes` stops every run after one milestone, so you can look at each one. |
| `backlog=<0-10>` | 3 | A backlog milestone runs after this many milestones; 0 only at the end of a stage. |
| `e2e=<every\|backlog>` | every | When all end-to-end tests run. `every` at the close of every milestone; `backlog` only at backlog milestones and at the end of a stage, while an ordinary milestone runs just the end-to-end tests of its own criteria. Worth it when the end-to-end tests take long. |

For a single run, `/wcc:milestone --once` stops after one milestone and
`--all` runs on despite `once=yes`. `/wcc:milestone --parallel <n>` sets the
parallelism like `/wcc:options` does.

## After a plugin update

The project rules in your project's `CLAUDE.md` are a copy made when the
project started, and the planning documents follow the templates of that
time. A newer plugin may look for rules, sections or keys they do not have
yet. Run `/wcc:upgrade` in each project after updating the plugin;
`/wcc:status` and `/wcc:milestone` also tell you when a project is behind.

It reads the version the project was written with from the line
`wcc version:` in `CLAUDE.md`, shows what it would change and asks once
before it writes. It keeps what belongs to your project: build and test
commands, your own rules, every decision. What needs an answer from you,
such as a new section of the spec, it leaves to the command that asks for
it and names that command. `/wcc:upgrade --check` only shows what would
change.

## Commands

| Command | When | What it does |
|---------|------|--------------|
| `/wcc:idea <idea>` | start | Interview, then `IDEA.md` |
| `/wcc:spec` | after the interview | Stack, feasibility check, `SPEC.md`, `ARCHITECTURE.md`, `ROADMAP.md`, summary for approval |
| `/wcc:prepare` | after the approval, on a new machine or checkout | Machine, accounts and secrets checked, git repository, scaffold, working build and test commands |
| `/wcc:milestone` | to build | Milestones one after another, test-first, until done or the run needs you |
| `/wcc:feedback [what you noticed]` | any time | Without text: open manual checks with test guides. With text: a finding, sorted with you and recorded for the next run |
| `/wcc:change [change]` | any time | A change worked into all documents, the next stage planned, or the open decisions answered |
| `/wcc:status` | any time | Where the project stands and which command comes next |
| `/wcc:options` | any time | Show or set how runs behave |
| `/wcc:upgrade [--check]` | after a plugin update | Project files brought up to the installed plugin version |
| `/wcc:prototype <what to try>` | optional, before the interview | Throwaway prototype to find out what you want |

## What wcc puts in your project

- `CLAUDE.md` in the project root: the rules every Claude session in the
  project follows, and the build and test commands. If the project already
  had one, the rules are appended under `## wcc rules`.
- `docs/wcc/`: the planning documents. `IDEA.md` is yours; `SPEC.md`,
  `ARCHITECTURE.md`, `ROADMAP.md`, `UI-SPEC.md` (for projects with a user
  interface) and `EXPERIMENTS.md` describe the plan; `STATUS.md`,
  `DECISIONS.md` and `BACKLOG.md` track the state with the open decisions
  waiting for you, every decision taken without you, and the findings for
  later; `BACKLOG-DONE.md` keeps the findings worked off or dropped, moved
  there by each backlog milestone; `plans/` holds Claude's detailed
  plan per milestone. You can read all of them, but you only need
  `IDEA.md`, `STATUS.md` and, now and then, `DECISIONS.md`.

The planning documents are not your project's documentation. If you asked
for user or developer documentation in the interview, the spec lists those
documents with their paths outside `docs/wcc/` and checkable criteria, for
example "every command in `--help` is described". Every milestone ends by
bringing them up to date with what was built, and the final check makes
sure they are complete and true.

## Language

The plugin does not set the language of the conversation: Claude talks to
you in the language you write in or have configured for yourself.
Everything it writes to the repository is English, the planning documents
included, so other people can work on the project.

## Status of the plugin

Work in progress. The steps from idea to the first milestones have run in a
real project; other parts have not been tried end to end yet.
`CLAUDE.md` in this repository is for developing the plugin: it explains
the design decisions behind the workflow, where to change what, and the
open points.
