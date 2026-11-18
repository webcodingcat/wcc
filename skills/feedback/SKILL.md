---
name: feedback
description: Walk the user through the open manual checks of a wcc project one at a time, preparing what can be prepared, and take what the user noticed while trying a wcc project by hand - a bug, something that looks wrong, the result of a manual check, a wish - with or without a screenshot, sort it with the user and record it where the next phase picks it up. Use this whenever the user reports a finding from manual testing, says "I noticed", "this does not work", "this looks wrong", "AUTH-03 fits" or "here is a screenshot" in a wcc project. Without arguments, also use it when the user asks "what do I have to test", "show the manual checks" or "how do I test this".
argument-hint: "[what you noticed]"
---

# feedback: manual checks and findings from the user

What the user noticed: $ARGUMENTS

All planning documents live in `.project/`; file names below are relative to
it.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in;
this skill does not set one. A call without text carries no language: then
use the configured one, or the language of the conversation so far, never
the language of the documents or of this skill, which are English.
Everything written to the repository is English, whatever language the user
reported in.

The user is here now; the run that fixes a finding later is not watched. So
ask what you need, and record each finding so completely that the run can
act on it without asking.

## 1. Read

Read STATUS.md, ROADMAP.md, UI-SPEC.md if it exists, and the folder of the
current phase in `phases/`. Read REQUIREMENTS.md, SPEC.md and ARCHITECTURE.md by
their headings if they have more than 300 lines.

If the status is earlier than `READY`, nothing is built yet: feedback on the
idea or the spec goes to `/wcc:spec` during `SPEC_REVIEW` and to
`/wcc:change` otherwise. Say so and stop.

If STATUS.md says `IN_PROGRESS`, ask once whether a `/wcc:phase` run is
going right now; the answer decides where findings go (step 5).

## 2. Without arguments: guided checks

Walk through the open manual checks one at a time: those of a phase
waiting in `MANUAL_CHECK` first, then the others under "Open manual checks"
in STATUS.md, each with its guide from `phases/<id>/CHECKS.md`. If no
check is open, say so and that findings can be reported with `/wcc:feedback
<what you noticed>`.

For each check:

1. Prepare what you can, so the user only judges what needs a human: start
   the program with the command of the project CLAUDE.md, create the data
   or state the setup names, open the page. Say what you prepared. Do what
   needs no decision and changes nothing outside the project; leave the
   rest to the user with the exact step.
2. Show the steps and what should happen, briefly: "This is what should
   happen. Does it?" If the next check shares the setup, say so in one line.
3. Take the answer in the user's words:
   - "fits", "yes", "ok" or nothing: fits.
   - a description of what is different: a finding (step 3).
   - "skip": stays open.
   - "not now, because ...": stays open, with the reason as a note in
     `CHECKS.md`.
   - "would be nice if ...": a wish (step 3).
4. Record the result right away (step 5), so a session that ends early
   loses nothing, and go on to the next check.

A phase that changed start-up, migration or seed code gets one extra
check first: stop everything, start from scratch, see that it comes up and
shows real data.

After the last check, sort the findings collected on the way (step 4).

## 3. Understand a finding

Split a report into findings, one per problem. For each:

- what the user did, what happened, what they expected
- which requirement it concerns (`AUTH-03`), if any; a check that does
  not fit is a finding on its requirement
- screenshots and other images: the run cannot see them, so describe in
  words everything that matters (the screen, the element, what is wrong,
  any text shown). An image file on disk is also copied to
  `phases/F<n>/feedback/<short-name>.<ext>` once the `F` phase is known.

Look at the code and REQUIREMENTS.md to locate the cause; run the program or a test
if that settles it quickly. Fix nothing here: fixes are built test-first by
the run. If the description is not enough to reproduce the problem, ask, all
missing points in one question.

## 4. Sort and confirm

- **check ok**: an open check fits.
- **bug**: the project does not do what REQUIREMENTS.md or UI-SPEC.md says, or is
  plainly broken (a crash, lost data, an error the user cannot get past).
- **minor**: within the requirements, cosmetic or nice to have.
- **wish**: something the requirements do not say or say differently.

Present all findings as one question, one line each with class, reason and
destination, and let the user correct them. A minor finding is parked in
the backlog; offer once, for all of them, to fix chosen ones **soon** in the
feedback phase instead.

## 5. Record

If a run is going, write the confirmed findings to FEEDBACK.md (step 6) and
touch nothing else; the next `/wcc:phase` records them. A wish then waits
there too; tell the user to run `/wcc:change` once the run has stopped.
Otherwise record directly:

- **check ok**: `Result: fits` in its `CHECKS.md`, and remove its line from
  "Open manual checks".
- **bug of a phase waiting in `MANUAL_CHECK`**: add it to
  `phases/<id>/PLAN.md` under "Findings from manual testing", and set
  `Result: does not fit, see plan` in `CHECKS.md`. The next `/wcc:phase`
  fixes it on that branch and asks again.
- **other bugs, and minor findings wanted soon**: an `F` entry in
  ROADMAP.md. Add to an unchecked `F` phase whose `PLAN.md` has no plans yet;
  otherwise insert `- [ ] F<n> Feedback fixes` before the first entry of the
  current milestone that has not started (after a phase waiting in
  `MANUAL_CHECK`), or after the last entry if all are checked; numbers
  continue from the highest `F`. Write each finding into
  `phases/F<n>/PLAN.md` under "Findings", marked `bug` or `minor`, with
  what the user did, what happened, what the requirement expects, its ID,
  the description of any image and the path of any copied file. If STATUS.md
  says `DONE`, set `READY`.
- **other minor findings**: BACKLOG.md, `- [ ] minor: <component>: <what
  is wrong> (source: feedback)`. The backlog is a parking lot the user
  empties with `/wcc:change`.
- **wish**: offer to run `/wcc:change` now with the wish. If the user
  declines, add it to "Open decisions" in STATUS.md in the form the
  phase skill gives, with source `feedback`.

## 6. The queue

FEEDBACK.md exists only while findings wait for a running phase.
Create it if it is missing:

```
# FEEDBACK

Findings from manual testing, already sorted with the user by /wcc:feedback
while a phase was running. The next /wcc:phase records them at
its start and then deletes this file.

- <class>: <the finding, written as in section 5>
```

One line per finding (with indented details), in English, with its class
(`check ok`, `bug`, `minor`, `minor, soon`, `wish`) and, for a check, its ID.
Do not commit it.

## 7. Report

In at most five lines: which checks fit and which stay open, where each
finding went, and what comes next: `/wcc:phase` if something waits for
a run; nothing if a run is going, which picks the findings up by itself.
