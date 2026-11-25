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

All planning documents live in `.project/`. If that folder does not exist,
say that the project has not been started and that `/wcc:idea` begins it.

Read STATUS.md and ROADMAP.md, FEEDBACK.md if it exists, and the plan of the current phase in
`phases/<id>/PLAN.md` if there is one. Read STATUS.md by its first line,
"Current state", "Open manual checks" and "Open decisions", and of
ROADMAP.md the section of the current milestone. Then report in at most ten lines:

- the status and what it means
- progress: phases done and open in the current milestone, and plans done
  and open in the current phase
- what waits for the user: open manual checks (results go to
  `/wcc:report`), the reason for `BLOCKED`, plans marked `[!]` in the
  plan, entries under "Open decisions" in STATUS.md (answered with
  `/wcc:change`; name those `In clarification` and whether their result
  file in `decisions/` exists already)
- findings queued in FEEDBACK.md and open `F` phases, if any
- the backlog in one line, counted per severity, most severe first (for
  example `Backlog: 1 security, 2 bug, 3 later`), with `/wcc:change` to
  take items in; nothing if it is empty
- the wcc version, only if the line `wcc version:` in the project CLAUDE.md
  is missing or older than `version` in `.claude-plugin/plugin.json` two
  folders above this skill folder: say that `/wcc:upgrade` brings the
  project up to date, and that it comes before the next command
- the next command:

| Status | Next |
|--------|------|
| `IDEA` | `/wcc:idea` |
| `IDEA_DONE`, `SPEC_REVIEW` | `/wcc:spec` |
| `READY` | `/wcc:prepare`, then `/wcc:phase` |
| `IN_PROGRESS`, `BLOCKED` | `/wcc:phase` |
| `MANUAL_CHECK` | `/wcc:report`, then `/wcc:phase` |
| `DONE` | `/wcc:cleanup` while `phases/` holds folders of a finished milestone, then `/wcc:change` for the next milestone |

Every `/wcc:phase` builds one phase and stops. For `IN_PROGRESS`
between two phases, recommend starting it in a new session or after
`/clear`: the next phase reads the project files afresh and needs
nothing from the old context.

Check that the files agree with each other and name every contradiction, for
example `DONE` with unchecked phases, `IN_PROGRESS` without an approved
spec, or a checked phase whose plan has open plans. Do not repair
anything; say what is inconsistent and let the user decide.
