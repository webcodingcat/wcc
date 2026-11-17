---
name: upgrade
description: Bring an existing wcc project up to date with the installed version of the wcc plugin - the project CLAUDE.md, the planning documents in docs/wcc/ and their keys - so the skills of the newer version find what they look for. Use this whenever the user says "upgrade the project", "update to the new wcc version", "migrate the project", after updating the plugin, or when /wcc:status reports that the project was written by an older wcc version.
argument-hint: "[--check]"
---

# upgrade: bring a project up to the installed wcc version

Arguments: $ARGUMENTS

All planning documents live in `docs/wcc/`; file names below are relative to
it. Only the project CLAUDE.md sits in the project root.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in;
this skill does not set one. A call without text carries no language: then
use the configured one, or the language of the conversation so far, never
the language of the documents or of this skill, which are English. The
options quoted below are given in English: offer them in the user's
language. Everything written to the repository is English.

The skills of a newer wcc version search the project files for keys,
sections and rules that an older version did not write. A plugin cannot ship
a CLAUDE.md that is always loaded, so the project CLAUDE.md is a copy of a
template and never changes when the plugin does. This skill closes that gap.
It changes the form of the project files, never their decisions: what the
user decided stays decided, and what needs a new decision goes to the skill
that asks for it.

With `--check`, only report what would change, and write nothing.

## 1. Versions

- If `docs/wcc/STATUS.md` does not exist, say that this is not a wcc
  project, that `/wcc:idea` starts one, and stop.
- The installed version is `version` in `.claude-plugin/plugin.json`, two
  folders above this skill folder.
- The project version is the line `wcc version: <x.y.z>` near the top of
  the project CLAUDE.md. Projects started before version 0.16.0 have no such
  line: call their version "before 0.16.0".
- If the project version is newer than the installed one, say that the
  plugin is older than the project, recommend updating the plugin, and stop.
  Writing older rules over newer ones would undo them.

## 2. Is a run going?

This skill edits STATUS.md, ROADMAP.md and the project CLAUDE.md, which a
milestone run reads and writes. If the status is `IN_PROGRESS`, ask whether
a run is going in another session right now. If so, say to run the upgrade
after it ends, and stop.

If the project is a git repository, check that the files this skill will
touch have no uncommitted changes. If they have, show them and ask whether
to commit them first, include them, or stop. Work on the branch that is
checked out. On a milestone branch `wcc/m<n>` or `wcc/f<n>`, say that the
upgrade reaches the base branch with the merge of that milestone.

## 3. Collect

Read `references/migrations.md` in this skill folder. Check every entry, not
only the ones newer than the project version: each entry says how to detect
that it is still needed, and a project may have been upgraded by hand or
only in part. Collect each entry whose check finds something to do.

Then compare the project with the current templates, to catch what no entry
describes:

- **Project CLAUDE.md** against `../idea/assets/project-CLAUDE.md`. If the
  project had a CLAUDE.md before wcc, the rules sit under `## wcc rules`;
  compare only that part. Go section by section:
  - a rule of the template that is missing or worded differently in the
    project: take the template wording.
  - the section "Build and test commands": keep the project's content; it
    was written by `/wcc:prepare` for this project.
  - lines that name something of this project (a path, a command, a tool, a
    service, a requirement) or that no template would contain: they were
    added for this project. Keep them in the section they are in.
  - a line where you cannot tell whether it is an outdated template rule or
    a deliberate project rule: do not decide; put it in the list for the
    user in step 4.
- **Planning documents** against the templates in `../idea/assets/` and
  `../spec/assets/`: compare the section headings and the key terms (the
  line names in "Options", `Type`, `Cost of change`, `Stops for: manual
  check` and the like). A heading or key that the template has under a new
  name is renamed. A section that is missing and needs no content is added
  with the template's placeholder. A section that is missing and needs
  content, decided by the user or derived from IDEA.md, is not invented: it
  becomes a follow-up for the skill that writes it (`/wcc:change` for IDEA,
  SPEC, ARCHITECTURE and ROADMAP, `/wcc:prepare` for build and test
  commands).

Leave alone: the code, the content of entries (requirements, decisions,
backlog items, experiments), the plans of finished milestones, and
`decisions/`. Their form may be older, but nothing reads them by key any
more, and rewriting them would change the history of the project.

## 4. Confirm

Show one list, at most about fifteen lines, grouped:

- **Changes now:** each change in one line, with the version that brought it
  where known (for example "STATUS.md: add the test guide to 5 open manual
  checks (0.15.0)").
- **Follow-ups:** what needs a skill that asks the user, with the command
  (for example "SPEC.md has no section Documentation: `/wcc:change` add
  documentation (0.13.0)").
- **Your call:** the lines from step 3 that could be an old rule or a
  project rule, each with your recommendation.

With `--check`, stop here. Otherwise ask once, with the options "apply",
"show the changes to the project CLAUDE.md first" and "cancel". The second
shows those changes as before and after, then asks again.

If nothing is to be done, set the version line if it is missing or older
(step 5), say that the project is up to date, and stop.

## 5. Apply

- Make the changes from step 4, with the user's answers for "Your call".
- Set the line `wcc version: <installed version>` in the project CLAUDE.md,
  right below its first heading (below `## wcc rules` if the rules sit
  there).
- Add an entry to DECISIONS.md:

  ```
  ## <date> Upgrade to wcc <installed version>
  
  - Type: approval
  - Phase: upgrade
  - What: project files brought from wcc <old version> to <installed version>: <changes in a few words>
  - Why: the skills of the installed version search for these keys and rules
  - Cost of change: low
  ```

- In a git repository, commit the changed files by name, never with
  `git add -A`, with the message `Upgrade wcc project files to <installed
  version>`.

## 6. Report

At most ten lines: from which version to which, what changed, the
follow-ups with their commands in the order to run them, and then the next
command for the status (as `/wcc:status` would name it).
