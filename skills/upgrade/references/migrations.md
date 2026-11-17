# Migrations

One entry per plugin version whose change needs something in an existing
project. Every entry says how to detect that it is still needed, so it can be
checked on any project, whatever version it claims. Entries only for
versions that need nothing in a project are left out.

Changes to rules in the project CLAUDE.md need no entry here: `/wcc:upgrade`
compares the project CLAUDE.md with the current template anyway. An entry is
needed when a planning document needs a new section, a renamed key, content
in a new form, or a follow-up with another skill.

Format:

```
## <version> <short title>

- Detect: <what in the project shows that the entry is still needed>
- Do: <what /wcc:upgrade changes now>
- Follow-up: <skill the user runs afterwards, and why; omit if none>
```

Newest entries first.

## 0.19.0 End-to-end tests selectable per criterion

- Detect: the option `Full e2e run` in STATUS.md is `backlog and stage end`,
  and an end-to-end test (from the end-to-end command of the project
  CLAUDE.md) carries the IDs of several criteria in one name instead of one
  subtest per criterion; and BACKLOG.md has no open item for it yet.
- Do: add one open item to BACKLOG.md: split the named end-to-end tests into
  subtests of one criterion each, under groups named after the component,
  keeping the shared setup (rule 4 of the `test-writer`). Name the tests.
  Do not change the tests themselves: that is code, and the next backlog
  milestone does it with tests and review.
- Follow-up: none; until the backlog milestone has run, the option saves
  little, because nearly every milestone selects the large tests.

## 0.18.0 Checked backlog items in BACKLOG-DONE.md

- Detect: BACKLOG.md has a checked item (`- [x]`).
- Do: create `BACKLOG-DONE.md` from the template
  `../../idea/assets/BACKLOG-DONE.md` (from this file) if it is missing, and
  move every checked item of BACKLOG.md there unchanged, with its notes and
  continuation lines, under one heading `## Before 0.18.0` (below any
  existing headings). Keep the order and leave the open items in
  BACKLOG.md. Moving is not rewriting: the content of the items stays as
  it is. Do not do this while a milestone is running.

## 0.17.0 Open decisions in STATUS.md

- Detect: STATUS.md has no section "Open decisions", or BACKLOG.md has an
  unchecked item marked `needs decision`.
- Do: add the section "Open decisions" after "Open manual checks", with the
  comment from the STATUS.md template. Move every unchecked BACKLOG.md item
  marked `needs decision` there, in the form given under "Open decisions"
  in the milestone skill (`../../milestone/SKILL.md` from this file): one
  question per entry, so split an item that mixes several, and leave the
  parts that need no decision in BACKLOG.md without the mark. Write
  context, options and a recommendation from the item, SPEC.md,
  ARCHITECTURE.md and the code; that is form, the answer stays with the
  user. Number the entries `Q1`, `Q2` and so on and keep the source of the
  item. This entry is the exception to "leave the content of backlog items
  alone". Write `- none` if nothing was moved.
- Follow-up: `/wcc:change` without arguments, if entries were moved: it
  shows them and takes the answers.

## 0.16.0 Version line in the project CLAUDE.md

- Detect: the project CLAUDE.md has no line `wcc version: <x.y.z>`.
- Do: nothing extra; step 5 of the skill sets the line.

## 0.15.0 Test guides for open manual checks

- Detect: an entry under "Open manual checks" in STATUS.md without the
  lines `Start`, `Steps` and `Expected`.
- Do: write the guide for each such entry, in the form given under "Manual
  criteria" in the milestone skill (`../../milestone/SKILL.md` from this
  file), from SPEC.md, UI-SPEC.md, the project CLAUDE.md and the code: the
  start command, the setup, the names on the screen, what to expect. Keep
  the text of the entry and any note it carries, for example on which
  milestone it is checked with. Replace the comment under the heading with
  the one from the STATUS.md template.

## 0.13.0 Documentation

- Detect: the status is `SPEC_REVIEW` or later and SPEC.md has no section
  "Documentation".
- Do: nothing in SPEC.md; which documentation the project needs is the
  user's answer, not an assumption.
- Follow-up: `/wcc:change` with "add documentation". It asks who needs
  documentation and adds the section to IDEA.md and SPEC.md. From then on
  every milestone ends with the task `Documentation`.

## 0.12.0 Option "Full e2e run"

- Detect: the section "Options" in STATUS.md has no line `Full e2e run`.
- Do: add `- Full e2e run: every milestone`, the default, which keeps the
  behaviour the project had.
- Follow-up: if the project has end-to-end tests but "Build and test
  commands" in the project CLAUDE.md lists no separate command for them,
  `/wcc:prepare` separates it. Without it the option has no effect.

## 0.11.0 Section "Options"

- Detect: STATUS.md has no section "Options".
- Do: create it right after "Current state" with the lines and defaults of
  the STATUS.md template. If "Current state" has a line `Parallel agents`,
  move it into the new section with its value: the user chose it. Say in
  the report that the default is 1 since 0.11.0 (it was 3), and that
  `/wcc:options` changes it.

## 0.10.0 Planning documents in English

- Detect: the planning documents or the project CLAUDE.md are written in
  another language, for example German headings in IDEA.md or SPEC.md.
- Do: translate the project CLAUDE.md, all planning documents and the plans
  of milestones that are not finished. Keep the meaning, not the wording.
  Use the headings and key terms of the current templates exactly, because
  the skills search for them. Keep identifiers as they are: `R1.2`, `M3`,
  `B1`, `F2`, file names, branch names, commands, names of code. Leave
  `decisions/` and the plans of finished milestones as they are. This is
  the largest migration; list it as one change with the number of files.
