# Project rules (wcc)

wcc version: <set by /wcc:idea>

These rules come from the wcc plugin. After the plugin is updated,
`/wcc:upgrade` brings them and the planning documents up to date.

## Language

- The language of the conversation is not set here; it follows the person
  working with Claude.
- Everything in the repository is English: code, code comments, commit
  messages, repository docs and the planning documents in `docs/wcc/` (IDEA,
  SPEC, ARCHITECTURE, UI-SPEC, ROADMAP, STATUS, DECISIONS, BACKLOG,
  BACKLOG-DONE, EXPERIMENTS, plans/, decisions/). Other people may work on the project.

## Documents

- All planning documents live in `docs/wcc/`.
- IDEA.md is the only source for goals and non-goals. Changes to the idea go
  into IDEA.md first; SPEC, ARCHITECTURE and ROADMAP are then updated from it.
- Read `docs/wcc/STATUS.md` at the start of every session and update it at
  the end.
- The planning documents are not the project's documentation. User and
  developer documentation lives where "Documentation" in SPEC.md says. It
  describes what is built, never the target vision, and is updated by the
  documentation task at the close of each milestone. Other tasks do not edit
  it, so parallel tasks do not collide on it.

## During a milestone run

These rules apply while `/wcc:milestone` is running, because nobody is
watching then. In a normal conversation the user is present: ask when
something is unclear.

- Ask no questions. If something is unclear, decide, record it in
  DECISIONS.md with its "Cost of change" and continue.
  Pick the simplest solution that fits SPEC.md. This covers decisions whose
  consequences are low or medium. A decision with high consequences (data
  schema, file or save formats, public URLs and API shapes, authentication,
  anything that later needs a migration or breaks clients) is not yours to
  take: if SPEC.md, ARCHITECTURE.md and DECISIONS.md do not settle it, the
  task waits for the user. This should be rare, because the spec and the
  planning of each milestone are meant to settle these beforehand.
- Repairing and adding is allowed, changing and replacing is not:
  - Allowed without stopping, each recorded in DECISIONS.md: fixing a bug;
    adding what correct behaviour needs but SPEC.md does not spell out
    (input validation, error handling, an empty state); fixing what blocks
    the task (a missing import, broken build configuration); extending an
    interface from ARCHITECTURE.md without breaking it (an additional return
    value, an optional parameter). The main session then updates
    ARCHITECTURE.md.
  - Not allowed: renaming or removing something in an interface, changing
    its parameters or meaning, adding or dropping a component, replacing a
    library or another part of the stack, and anything that contradicts an
    acceptance criterion or makes two of them contradict each other.
  - When in doubt, it is not allowed.
- Stay within the task. Fix a problem in other code only if it blocks the
  task; otherwise report it for BACKLOG.md.
- Never build a way into the product or the tests for debugging: no debug
  script, endpoint, port, account, key, shell access, hidden command or
  encoded payload. Temporary aids stay outside the repository and are
  never committed. A protection the project builds (access control, an
  isolated component, a network boundary) is never worked around, not even
  temporarily. If a cause cannot be found without such access, the task
  stops and reports it.
- A task cannot be finished, and must not be forced, when:
  - something destructive or irreversible would be needed
  - a decision with high consequences is needed that nothing settles yet
  - a change to SPEC.md or ARCHITECTURE.md that is not allowed above becomes
    necessary
  - its tests, the linter or the build still fail after three repair attempts
  - a credential, an account or a secret is missing (401, 403, "not logged
    in", an unset environment variable). This is not a bug. Do not work
    around it with fake values or mocks.
  - administrator or root rights are missing ("access denied", "requires
    elevation"). This is not a bug either. Never try to obtain them (`sudo`,
    `runas`, an elevated shell) and never weaken a test to get around them.
  - a package cannot be installed or does not exist under the planned name.
    Never try a similar name instead: a near miss may be a malicious package.
- Such a task does not end the run at once. It is marked in the plan, the
  tasks depending on it are skipped, everything else is finished, and only
  then STATUS.md is set to `STATUS: BLOCKED` with the reason and the exact
  step needed from the user.
- Subagents do not write DECISIONS.md, BACKLOG.md, STATUS.md, ROADMAP.md,
  FEEDBACK.md or the plans. FEEDBACK.md holds findings `/wcc:feedback`
  queued while a run was going; the main session records them before the
  next milestone. They return decisions and findings; the main session writes them
  down.

## Git during a milestone run

Several agents work in worktrees of the same repository, so:

- Stage files by name. Never `git add -A` or `git add .`: they sweep up
  files that belong to nobody's task.
- Never `git stash`: the stash is shared across all worktrees.
- Never `git reset --hard`, `git clean`, a forced push, or a checkout that
  discards changes.
- A subagent commits only on the branch of its own task.

## Secrets

- Secrets live in `.env`, which is never committed. `.env.example` lists the
  names of the variables without values.
- Never read, print or log the content of `.env`, and never write a secret
  into code, tests, planning documents or commit messages.

## Always

- Build only what the current milestone needs. Write no code in advance for the
  target vision; only the interfaces must allow it.
- Prefer simple code and a simple project layout.

## Tests

- Every acceptance criterion marked `auto` or `browser` gets at least one test
  before implementation.
- A task is done only when its own tests, the other tests of the packages or
  modules it changed, the linter on those and the build pass. A milestone is
  done only when the full test suite and the linter over the whole project
  pass. Which end-to-end
  tests belong to it is set by the option `Full e2e run` in
  `docs/wcc/STATUS.md`.
- Tests that need expensive setup (a running service, a VM, a database, a
  logged-in browser session) share one instance unless they test the setup
  itself, also across components: one instance for the whole run, not one
  per group. Checks that need an expensive step (a restart, a recreation)
  share one run of it. Independent subtests that mostly wait (variants per
  image or platform, a VM booting) run in parallel, within the capacity of
  the environment. Each criterion still has a test or subtest of its own
  that carries its ID and no other, so the test filter can select it
  alone. No fixed waits in tests; wait for a condition with a deadline.
- Test files live next to the code under test or in the test directory,
  never in `docs/` or other folders for documentation, assets or
  configuration. A test that checks a document reads it by its path.
- Change a test only if it contradicts SPEC.md, and record that in DECISIONS.md.
  Never weaken a test to make it pass.

## Reviews

- Reviewers check against SPEC.md and the acceptance criteria. Decisions in
  ARCHITECTURE.md and DECISIONS.md are not up for debate.
- Fix only blockers immediately: bug, violation of SPEC.md, security problem,
  failing test. What would need a change to SPEC.md or ARCHITECTURE.md goes
  to "Open decisions" in `docs/wcc/STATUS.md`, for the user. Everything
  else goes to BACKLOG.md.
- At most one review round per task. Small tasks of one component that the
  plan groups share one review.

## Build and test commands

<filled in by /wcc:prepare>
