---
name: prepare
description: Check that the machine has everything a wcc project needs and set the project up so the milestones can run - git repository, stack tools, accounts and secrets, dependencies, project scaffold, working build and test commands. Use this after the spec is approved, on a fresh checkout or a new machine, or whenever the user says "prepare the project", "check my system" or "set up the project".
---

# prepare: check the machine and set up the project

The unattended run must not fail at night because a tool is missing. This
skill finds that out while the user is present. It is repeatable: whatever
already exists is only checked, so it also serves a fresh checkout or a newly
installed machine.

All planning documents live in `docs/wcc/`; file names below are relative to it.

## Preconditions

SPEC.md exists and STATUS.md says `READY` or a later value. Otherwise point to
`/wcc:spec` and stop. This skill does not change STATUS.md.

## Language

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in. A
call without text carries no language: then use the configured one, or the
language of the conversation so far, never the language of the documents or
of this skill, which are English. Everything written to the repository is
English: code, configuration, README, commit messages and the planning
documents.

## Step 1: check the system

Read the stack from SPEC.md and check that every tool it needs is installed
and new enough: git, runtime, package manager, and the browser test tool if
any criterion is marked `browser`. Check by running the tool, not by guessing.

Read "Machine prerequisites" in SPEC.md and check each entry the
same way. If it names elevated rights, check whether this session has them
(on Windows for example `net session`, elsewhere `id -u`).

If something is missing, list it with the install command for the user's
operating system and stop. Do not install system software yourself: that
changes the user's machine and is their decision. The same holds for
elevated rights: do not try to obtain them. Tell the user that the
milestones need Claude Code started as administrator, what for, and that
every agent of the run then has those rights; then stop.

## Step 2: repository

If the folder is not a git repository, first check the project name. The
repository, package names and the scaffold make a later rename expensive, so
this is the last cheap moment. Ask the user, with the rename as
recommendation, if either holds:

- the heading of IDEA.md ends in `(working title)`
- the folder name is not the slug of the project name in IDEA.md

The options are "Rename the folder first" and "Keep it as it is". For a
rename: if there is no name yet, agree on one, update the headings of the
planning documents in `docs/wcc/`, and tell the user to close this session,
rename the folder and run `/wcc:prepare` again from a new session there;
then stop. If the user keeps it, record that in DECISIONS.md and continue.
Do not ask again when the folder is already a git repository.

Then run `git init`, add a `.gitignore`
fitting the stack and commit the existing files. Milestones need commits and
worktrees, so this is not optional. `.env` must be in `.gitignore`.

## Step 3: accounts and secrets

Read "Accounts and secrets" in SPEC.md. For every entry:

- make sure `.env.example` lists the variable name, without a value
- check that the variable is set, in `.env` or in the environment. Check only
  that it exists and is not empty. Never print, log or copy its value.
- where it can be done without side effects, check that the credential works,
  for example with a read-only call

If something is missing, tell the user exactly what to create and where to
put it, and stop. The user enters secrets themselves; do not ask for them in
the chat.

Add `Read(./.env)` and `Read(./.env.local)` to `permissions.deny` in the
project's `.claude/settings.json`, so no agent reads the secrets by accident
during a run.

## Step 4: project

- No project scaffold yet: create it as SPEC.md and ARCHITECTURE.md describe
  it: package manifest, test framework, linter, build, and one trivial test
  that passes. Use the versions named in SPEC.md. Add no feature code; that
  belongs to the milestones.
- Add `docs/wcc/experiments/` to `.gitignore`. It is the scratch folder of
  the feasibility check; `/wcc:spec` deletes it, and if an interrupted run
  left it behind, it must not be committed or built.
- Scaffold exists: install the dependencies.

If a package cannot be installed or does not exist under the name in SPEC.md,
stop and tell the user. Never try a similar name: a near miss may be a
malicious package.

## Step 5: prove it

Run build, linter and tests. All three must succeed. Then:

- write the exact commands into the section "Build and test commands" of the
  project CLAUDE.md. If some tests need the real environment or a browser
  and take long (end-to-end tests), give them a command of their own, separate
  from the fast tests, and say so in the line; `/wcc:milestone` uses it to
  choose which end-to-end tests run (option `Full e2e run`). Name in the line
  how to append a filter by test name to it (for example `-run <regexp>`).
  For the fast tests and the linter, also name how to run them on one
  package, module or directory (for example `go test ./internal/store/...`):
  a milestone checks each task that way and runs the full commands only at
  its close
- write a section "Prerequisites and setup" into the project README, so
  someone without this plugin can set up their machine too
- commit

## Step 6: report

Tell the user in at most five lines what was checked, what was created, and
that the next step is `/wcc:milestone`.
