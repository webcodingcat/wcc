---
name: status
description: Show where a wcc project stands and which command comes next. Use this whenever the user asks "where are we", "what is the status", "what comes next", "what do I have to do now" or returns to a wcc project after a break.
---

# status: where the project stands and what comes next

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in. A
call without text carries no language: then use the configured one, or the
language of the conversation so far, never the language of the documents or
of this skill, which are English. This skill only reads; it changes no file.

All planning documents live in `docs/wcc/`. If that folder does not exist,
say that the project has not been started and that `/wcc:idea` begins it.

Read STATUS.md and ROADMAP.md, FEEDBACK.md if it exists, and the plan of the current milestone in
`plans/` if there is one. Then report in at most ten lines:

- the status and what it means
- progress: milestones done and open in the current stage, and tasks done and
  open in the current milestone
- what waits for the user: open manual checks (results go to
  `/wcc:feedback`), the reason for `BLOCKED`, tasks marked `[!]` in the
  plan, entries under "Open decisions" in STATUS.md (answered with
  `/wcc:change`; name those `In clarification` and whether their result
  file in `decisions/` exists already), and in older projects backlog items marked
  `needs decision`
- findings queued in FEEDBACK.md and open `F` entries, if any
- the options from the section "Options" in STATUS.md in one line, only
  where they differ from the defaults (`/wcc:options` changes them)
- the wcc version, only if the line `wcc version:` in the project CLAUDE.md
  is missing or older than `version` in `.claude-plugin/plugin.json` two
  folders above this skill folder: say that `/wcc:upgrade` brings the
  project up to date, and that it comes before the next command
- the next command:

| Status | Next |
|--------|------|
| `IDEA` | `/wcc:idea` |
| `IDEA_DONE`, `SPEC_REVIEW` | `/wcc:spec` |
| `READY` | `/wcc:prepare`, then `/wcc:milestone` |
| `IN_PROGRESS`, `MANUAL_CHECK`, `BLOCKED` | `/wcc:milestone` |
| `DONE` | `/wcc:change` for the next stage |

Every `/wcc:milestone` builds one milestone and stops. For `IN_PROGRESS`
between two milestones, recommend starting it in a new session or after
`/clear`: the next milestone reads the project files afresh and needs
nothing from the old context.

Check that the files agree with each other and name every contradiction, for
example `DONE` with unchecked milestones, `IN_PROGRESS` without an approved
spec, or a checked milestone whose plan has open tasks. Do not repair
anything; say what is inconsistent and let the user decide.
