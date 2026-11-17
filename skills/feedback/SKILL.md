---
name: feedback
description: Show the open manual checks of a wcc project with a guide how to test each one, and take what the user noticed while trying a wcc project by hand - a bug, something that looks wrong, the result of a manual check, a wish - with or without a screenshot, sort it with the user and record it where the next milestone run picks it up. Use this whenever the user reports a finding from manual testing, says "I noticed", "this does not work", "this looks wrong", "R1.2 fits" or "here is a screenshot" in a wcc project. Without arguments, also use it when the user asks "what do I have to test", "show the manual checks" or "how do I test this".
argument-hint: "[what you noticed]"
---

# feedback: record what the user found by hand

What the user noticed: $ARGUMENTS

All planning documents live in `docs/wcc/`; file names below are relative to it.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in;
this skill does not set one. A call without text carries no language: then
use the configured one, or the language of the conversation so far, never
the language of the documents or of this skill, which are English.
Everything written to the repository is English, so a finding is recorded in
English, whatever language the user reported it in.

The user is here now, and the milestone run that fixes the finding later is
not watched. So this skill asks what it needs to know, and records each
finding so completely that the run can act on it without asking.

## 1. Read

Read STATUS.md, ROADMAP.md, BACKLOG.md, UI-SPEC.md if it exists, and the
plan of the current milestone in `plans/`. SPEC.md and ARCHITECTURE.md grow
large; if one has more than 300 lines, list its headings with Grep and read
the sections the findings concern, as the section "Reading the planning
documents" of the milestone skill (`../milestone/SKILL.md`) describes.

If the status is earlier than `READY`, nothing is built yet: feedback on the
idea or the spec goes to `/wcc:spec` during `SPEC_REVIEW` and to
`/wcc:change` otherwise. Say so and stop.

## Without arguments: show the open checks

If the user gave no text and no image, show every entry under "Open manual
checks" in STATUS.md, numbered, each with its test guide: start, setup,
steps and what to expect. Put the checks of a milestone waiting in
`MANUAL_CHECK` first and say that the run waits for them. If several checks
share the start and setup, say that once.

An entry without a guide (written before version 0.15.0) gets one now,
from SPEC.md, UI-SPEC.md, the project CLAUDE.md and the code. Show it, but
do not write it to STATUS.md if a run may be going (see 4).

Then ask for the results, for example "1 and 3 fit, at 2 the animation
flickers", with screenshots if helpful. Take the answer as the user's
report and continue with step 2. If no check is open, say so and that
findings of their own can be reported with `/wcc:feedback <what you
noticed>`.

## 2. Understand

Split the report into findings, one per problem. For each one, find out:

- what the user did, what happened, what they expected
- which acceptance criterion it concerns, if any (`R<n>.<m>`). A finding
  can also be the result of an open manual check in STATUS.md, "fits" or
  "does not fit".
- **Screenshots and other images:** look at them closely. The run cannot
  see images from this conversation, so describe in words everything that
  matters: the screen, the element, what is wrong with it, any text or
  error shown. If the user named an image file on disk, also copy it to
  `feedback/<short-name>.<ext>`.

Look at the code and SPEC.md to locate the cause; run the app or the tests if
that settles a question quickly. Do not fix anything here: fixes are built
test-first by the run. If the description is not enough to reproduce the
problem, ask, all missing points in one question.

## 3. Sort and confirm

Sort each finding:

- **check ok**: an open manual check fits.
- **bug**: the project does not do what SPEC.md or UI-SPEC.md says, or it is
  plainly broken (a crash, lost data, an error the user cannot get past). A
  check that does not fit is a bug of its criterion.
- **minor**: within SPEC.md, but cosmetic or nice to have.
- **wish**: the user wants something SPEC.md does not say or says
  differently.

Present the findings as one question: one line each with the class, the
reason and where it will go (see 5). Let the user correct it. Confirming a
list is faster than a question per finding, and nothing is sorted without
the user seeing it.

A minor finding waits in the backlog for the next backlog milestone. Say so
in its line, and offer to fix it **soon** instead: it then goes into the
feedback milestone like a bug. Offer this once for all minor findings, not
per finding; the user names the ones they want soon.

## 4. Is a run going?

If STATUS.md says `IN_PROGRESS`, a `/wcc:milestone` run may be working in
this checkout right now, on its milestone branch. Writing ROADMAP.md or a
plan under its feet would end up in its commits or collide with them. Ask
whether a run is going, in the question of step 3.

- A run is going: write the confirmed findings to FEEDBACK.md, the queue
  (see 6), and touch nothing else. The next `/wcc:milestone` takes them up
  at its start. A wish cannot go to `/wcc:change` now, because that edits
  SPEC.md under the run's feet: queue it as `wish`. The run adds it to
  "Open decisions" in STATUS.md; tell the user to run `/wcc:change` once
  the run has stopped.
- Otherwise, and for every other status: record them directly (see 5).

## 5. Record

- **check ok**: remove the check from "Open manual checks" in STATUS.md.
- **bug in a milestone waiting in `MANUAL_CHECK`** (a check of that
  milestone that does not fit, or a bug in what only it builds): add it to
  its plan `plans/M<n>.md` under "Findings from manual testing" and leave
  its check open in STATUS.md with ` - does not fit, see plan`. The next
  `/wcc:milestone` fixes it on that branch and asks again.
- **other bugs, and minor findings the user wants soon**: a feedback milestone `- [ ] F<n> Feedback fixes` in
  ROADMAP.md. If an unchecked `F` entry without tasks in its plan exists,
  add to it. Otherwise insert a new one before the first unchecked entry of
  the current stage that has not started (after the milestone waiting in
  `MANUAL_CHECK`, if there is one), or after the last entry of the last stage
  if every entry is checked; numbers continue from the highest `F`. Write
  each one into `plans/F<n>.md` under "Findings", marked `bug` or `minor`:
  what the user did, what happened, what SPEC.md expects or the user wants
  instead, the criterion ID, the description of any image and the path of
  any copied file. If STATUS.md says `DONE`, set it to
  `READY`, because the roadmap has an open entry again.
- **other minor findings**: BACKLOG.md, `- [ ] <finding> (source: feedback)`.
- **wish**: it needs `/wcc:change`, which keeps all documents consistent.
  Offer to run it now with the wish as its argument; the user is here, so
  nothing has to wait. If they decline, add it to "Open decisions" in
  STATUS.md, in the form the milestone skill describes there, with source
  `feedback`, so it is not lost and shows up at the next `/wcc:milestone`.

## 6. The queue

FEEDBACK.md exists only while findings wait for a running milestone run.
Create it if it is missing:

```
# FEEDBACK

Findings from manual testing, already sorted with the user by /wcc:feedback
while a milestone run was going. The next /wcc:milestone records them at
its start and then deletes this file.

- <class>: <the finding, written as in section 5>
```

Append one line per finding (with indented lines for details), in English,
with its class (`minor, soon` for a minor finding the user wants fixed
soon, `wish` for a wish) and, for a check, its ID. Do not
commit it: the run stages files by name, so the file stays out of its
commits until it is processed.

## 7. Report

In at most five lines: where each finding went, and what comes next. If
something waits for a run, the next command is `/wcc:milestone`; if a run
is going, it picks the findings up by itself.
