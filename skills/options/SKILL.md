---
name: options
description: Show and set the options of a wcc project that control how /wcc:milestone runs - how many agents work in parallel, how often a backlog milestone runs, how often the full end-to-end tests run. Use this whenever the user wants to change or see these settings, says "set the parallelism", "fewer agents", "e2e takes too long" or "wcc options", instead of editing STATUS.md by hand.
argument-hint: "[parallel=<1-5>] [backlog=<0-10>] [e2e=<every|backlog>]"
---

# options: how the milestone runs behave

Arguments: $ARGUMENTS

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in;
this skill does not set one. A call without text carries no language: then
use the configured one, or the language of the conversation so far, never
the language of the documents or of this skill, which are English. All
planning documents live in `docs/wcc/`; file names below are relative to it.

The options live in the section "Options" of STATUS.md and not in this
conversation or in the user's settings. `/wcc:milestone` reads STATUS.md
at the start of every run, so a value set here holds for every following
run until it is changed again, also after an interruption.

Every run implements one milestone and stops; there is no option to chain
them. A user who asks for that ("run all milestones", "do not stop after
each one") gets this answer: each milestone starts by hand, best in a new
session, because one session over several milestones grows its context and
uses up the usage limits faster without helping the work.

## The options

| Key | Line in STATUS.md | Values | Default | Effect |
|-----|-------------------|--------|---------|--------|
| `parallel` | `Parallel agents` | 1 to 5 | 1 | How many subagents `/wcc:milestone` runs at the same time. 1 runs everything one after another. More finish sooner but use up the usage limits of the account just as much faster. |
| `backlog` | `Backlog milestone after` | 0 to 10 | 3 | After how many milestones in a row a backlog milestone runs. 0 means only at the end of a stage. |
| `e2e` | `Full e2e run` | `every milestone`, `backlog and stage end` (short: `every`, `backlog`) | `every milestone` | When the close of a milestone runs all end-to-end tests. With `backlog and stage end`, an ordinary milestone runs only the end-to-end tests of its own criteria and of the files it changed; all of them run at the close of every backlog milestone and of the last entry of a stage. Saves time when the end-to-end tests take long, at the price that a regression in an older flow shows up only there. Has no effect if the project CLAUDE.md lists no separate command for end-to-end tests. |

The line names are keys that `/wcc:milestone` searches for. Keep them
exactly as written.

## Steps

1. If `docs/wcc/STATUS.md` does not exist, say that the project has not been
   started, that `/wcc:idea` begins it, and stop.
2. Read STATUS.md. If it has no section "Options" (projects started before
   the section existed), create it right after "Current state" with the
   defaults. If "Current state" still has a line `Parallel agents`, move it
   into the new section with its value; it is the value the user chose. A
   line `Stop after each milestone` (before version 0.25.0) has no effect
   any more: leave it to `/wcc:upgrade` and do not show it.
3. Arguments were given: read them as key and value, in the form above or
   in plain words ("two agents", "backlog only at the end"). Check every
   value against the table. If one is not allowed, say which values are and
   change nothing; the user is still there to correct it.

   No arguments: show the current values in a short table with what each
   one does, then ask which to change, offering the allowed values and the
   current one as options, in the user's language. If the user changes
   nothing, stop.
4. Write the new values to the section "Options". Change nothing else in
   STATUS.md and no other file.
5. Confirm in one or two lines what is set now and when it takes effect: at
   the next `/wcc:milestone`. A run that is going on in another session
   keeps the values it read at its start.
