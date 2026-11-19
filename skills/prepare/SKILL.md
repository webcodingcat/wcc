---
name: prepare
description: Check that the machine has everything a wcc project needs and set the project up so the phases can run - git repository, stack tools, accounts and secrets, dependencies, project scaffold, working build and test commands. Use this after the spec is approved, on a fresh checkout or a new machine, or whenever the user says "prepare the project", "check my system" or "set up the project".
---

# prepare: check the machine and set up the project

A phase must not fail halfway because a tool is missing. This
skill finds that out while the user is present. It is repeatable: whatever
already exists is only checked, so it also serves a fresh checkout or a newly
installed machine.

All planning documents live in `.project/`; file names below are relative to it.

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

The project folder and SPEC.md are the only sources. Other folders on the
machine, such as an older copy or a sibling project, hold decisions nobody
approved for this project; do not search or read them.

## Step 1: check the system

Read the stack from SPEC.md and check that every tool it needs is installed
and new enough: git, runtime, package manager, and the browser test tool if
the stack names one. Check by running the tool, not by guessing.

Read "Machine prerequisites" in SPEC.md and check each entry the
same way. If it names elevated rights, check whether this session has them
(on Windows for example `net session`, elsewhere `id -u`).

If something is missing, list it with the install command for the user's
operating system and stop. Do not install system software yourself: that
changes the user's machine and is their decision. The same holds for
elevated rights: do not try to obtain them. Tell the user that the
phases need Claude Code started as administrator, what for, and that
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
planning documents in `.project/`, and tell the user to close this session,
rename the folder and run `/wcc:prepare` again from a new session there;
then stop. If the user keeps it, record that in DECISIONS.md and continue.
Do not ask again when the folder is already a git repository.

If SPEC.md does not name the module or package name, ask the user now and
add it under "Stack".

Then run `git init`, add a `.gitignore`
fitting the stack and commit the existing files. Phases need commits and
worktrees, so this is not optional. `.env` must be in `.gitignore`.

## Step 3: accounts and secrets

Read "Accounts and secrets" in SPEC.md. For an environment variable:

- make sure `.env.example` lists the variable name, without a value; create
  `.env.example` only if there is at least one variable
- check that the variable is set, in `.env` or in the environment. Check only
  that it exists and is not empty. Never print, log or copy its value.

For the login of a tool, check it with a read-only call through the tool
itself, in the context SPEC.md names; never read the tool's credential
file.

Where it can be done without side effects, check that a credential works,
for example with a read-only call. An optional entry that is missing is
reported, not a reason to stop.

If something is missing, tell the user exactly what to create and where to
put it, and stop. The user enters secrets themselves; do not ask for them in
the chat.

Add `Read(./.env)` and `Read(./.env.local)` to `permissions.deny` in the
project's `.claude/settings.json`, so no agent reads the secrets by accident
during a run.

For the access agents need against a running instance through a tool's
login, add an allow rule for exactly that tool and context to
`permissions.allow` there (for example `Bash(hcloud --context <name> *)`).
Without it the permission check of an unattended run may refuse writes
against what looks like an unknown service. Never allow the tool as a whole.
In auto mode the classifier can still refuse such writes, and it reads
trusted infrastructure only from the user's own settings, never from the
project. So give the user a ready entry for `autoMode.environment` in
`~/.claude/settings.json`, one sentence naming the context, the service
and that it is a local development instance (for example "The hcloud
context `<name>` points at a local development cloud on this machine;
creating and deleting resources there is routine test work."), and one
more for the command that starts and stops that instance, if the project
has one, with its path and its subcommands. The user
adds it; do not write that file. `claude auto-mode config` shows whether
it is in effect.

## Step 4: project

- No project scaffold yet: create it as SPEC.md and ARCHITECTURE.md describe
  it: package manifest, test framework, linter, build, and one trivial test
  that passes. Use the versions named in SPEC.md. Add no feature code; that
  belongs to the phases.
- Add `.project/research/experiments/` to `.gitignore`. It is the scratch folder of
  the feasibility check; `/wcc:spec` deletes it, and if an interrupted run
  left it behind, it must not be committed or built.
- Scaffold exists: install the dependencies.
- Every reference under "References" in SPEC.md is readable at the path
  named there; if the setup fetches its copy, fetch it and write the path
  there.

If a package cannot be installed or does not exist under the name in SPEC.md,
stop and tell the user. Never try a similar name: a near miss may be a
malicious package.

## Step 5: prove it

Run build, linter and tests. All three must succeed. Then:

- write the exact commands into the section "Build and test commands" of the
  project CLAUDE.md. If some tests need the real environment or a browser
  and take long (end-to-end tests), give them a command of their own, separate
  from the fast tests, logging each test as it starts and ends, with a time
  limit of about twice the expected run, and say so in the line; `/wcc:phase` uses it to
  choose which end-to-end tests run. Name in the line
  how to append a filter by test name to it (for example `-run <regexp>`).
  For the fast tests and the linter, also name how to run them on one
  package, module or directory (for example `go test ./internal/store/...`):
  a phase checks each plan that way and runs the full commands only at
  its close
- write a section "Prerequisites and setup" into the project README, so
  someone without this plugin can set up their machine too
- commit

## Step 6: report

Tell the user in at most five lines what was checked, what was created, and
that the next step is `/wcc:phase`.
