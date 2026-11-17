---
name: change
description: Apply a change request to a wcc project consistently across IDEA.md, SPEC.md, ARCHITECTURE.md and ROADMAP.md, plan the next stage from the target vision once a stage is done, or take the user's answers to the open decisions a milestone run collected. Use this whenever the user wants to change the stack, add or drop a feature, add a tool, extend the target vision, start the next stage or decide the open decisions of a project that already has an IDEA.md, instead of editing a single file by hand.
argument-hint: "[what should change]"
---

# change: work a change into all documents

The change: $ARGUMENTS

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in. A
call without text carries no language: then use the configured one, or the
language of the conversation so far, never the language of the documents or
of this skill, which are English. Write the planning documents in English,
like everything else in the repository. All planning documents live in
`docs/wcc/`; file names below are relative to it.

## Open decisions

A milestone run collects what only the user can decide under "Open
decisions" in STATUS.md, each entry with a question, options and a
recommendation. The user is here now, so this is where they are answered.

- **No arguments:** if entries are open, present them all as one question:
  per entry its ID, the question, the options and the recommendation,
  offered in the user's language. The user may answer some and defer the
  rest. Each answer is a change: work it in with the steps below and remove
  its entry. If no entry is open and no change was named, ask what should
  change.
- **With a change that starts the next stage:** if entries are open, show
  them first in the same way. The next stage is planned on top of the
  answers; deferred entries stay open.
- **Any other change:** if it settles an open entry, remove that entry too.

An answer the user gives in their own words can differ from every option;
take it as it is. Record each answer in DECISIONS.md with its `Q<n>`.

## Asking

Ask the way the interview of `/wcc:idea` does: two to four options, one
marked as your recommendation, rounds of at most three questions, with the
AskUserQuestion tool if it is available. This holds for the open decisions
and for every question the change itself raises. Every question also offers:

- "I don't know": take the recommendation, record it in DECISIONS.md as
  `Type: assumption`, and work it in.
- "I need more details for this (separate session)": the question needs a
  discussion, see "Clarification sessions". Open decisions are often of
  this kind: they came up in a run, and the user meets them without the
  context they arose in.

## Clarification sessions

A long discussion would fill the context of this session, so it runs in a
child session, as in the interview. For each such question:

1. Note it on disk, so it is not forgotten: an open decision gets the line
   `  - In clarification: decisions/<topic>.md` under its entry in
   STATUS.md; another question goes under "Open decisions" as a new
   `Q<n>` entry with that line. Then continue with the other questions and
   work in what is answered.
2. Start a child session with `mcp__ccd_session__start_session` (Claude
   Code Desktop; load it with ToolSearch if it is only listed as deferred),
   with a fresh context and `use_worktree: false`, so it can write its
   result into this folder. The brief must stand on its own: the question
   with its context, options and recommendation, why it matters for this
   project, the paths of SPEC.md, ARCHITECTURE.md and of the code it
   concerns, and the instruction: talk to the user in their language;
   present the options with pros and cons for this project, look into the
   code and the documents where it helps, answer follow-up questions,
   decide nothing without the user. Once the user has decided, and not
   before, write the decision, the reasons and the rejected alternatives in
   English to `docs/wcc/decisions/<topic>.md`, change no other file, and
   tell the user in one line that they can return to `/wcc:change`.
3. If the session shows a card instead of starting, ask the user to start
   it. If no session tool is available, print the brief and ask the user to
   open a new session with it.

The result file is the signal that a clarification is finished, not a
message from the child session, which does not arrive reliably. So at every
call of this skill and after every round of questions, check the result
files of the entries in clarification. If one exists, read it, work the
decision in like an answer (steps below) and remove the entry. Entries
without a file stay open; a milestone run treats them like every other open
decision. If the user says a discussion is over but the file is missing,
ask for the decision here.

## Steps

1. Classify the change:
   - **Idea** (goal, non-goal, target vision, constraint): change IDEA.md
     first.
   - **Spec or architecture** (stack, component, requirement): IDEA.md stays as
     it is.
   - **Next stage** (a target vision item should be built now): write its
     requirements and acceptance criteria into SPEC.md, add a new section
     `## Stage <n>: <title>` with detailed milestones to ROADMAP.md
     before "Target vision (rough)" and remove the item from that rough list.
     Milestone numbers continue.
   - **Tooling for Claude** (for example an MCP server): besides the
     documents, it must be configured in the project so Claude Code can use it.
     Add one sentence to the project CLAUDE.md on what it is for.
2. Update every affected document: SPEC.md, ARCHITECTURE.md, UI-SPEC.md,
   ROADMAP.md and affected plans. A change to behaviour that is already
   documented names the affected documents in the milestone that builds it,
   so its documentation task updates them. A change to the documentation
   itself (a new audience, a new document, a project that had none) goes
   into "Documentation" in IDEA.md and SPEC.md; if it has to cover what is
   already built, schedule a milestone for it. If the change brings a new library, API or
   tool, let the `researcher` agent check it first, as in the spec. If the
   change replaces something an entry in EXPERIMENTS.md was checked with,
   set that entry to `superseded` with the reason; do not delete it. Never
   leave two documents contradicting each other, nor two places in one:
   search every document for the terms, names and values the change
   affects and update each place, because a rule is often stated in more
   than one section. Every milestone reads
   these documents, so add only what other components build on to
   ARCHITECTURE.md, not how a component does its work inside (see "Reading
   the planning documents" in `../milestone/SKILL.md`).
3. Record the change in DECISIONS.md with `Type: change`, `What` and `Why`
   in one sentence each.
4. State the cost honestly before applying it if code already exists:
   - additions are cheap and are scheduled as a new milestone or task
   - a stack change means rewriting existing code; name the affected
     milestones and schedule the rebuild as its own milestone
   - a target vision item the architecture cannot absorb needs a rebuild milestone
5. Remove the entries under "Open decisions" in STATUS.md this change
   settles, and BACKLOG.md items it takes up. Write `- none` if the section
   is empty then. In the plans, turn tasks marked `[!]` with ` - waits for
   Q<n>` of a settled entry back to `[ ]`, adjusted to the answer.
6. Update STATUS.md: if it was `BLOCKED` because of this point, or `DONE` and
   the roadmap now has open milestones again, set it to `READY`. Leave earlier
   values (`IDEA` to `SPEC_REVIEW`) as they are.
7. Summarize in at most five lines what changed and what happens next. If the
   change needs new tools on the machine, the next step is `/wcc:prepare`,
   otherwise `/wcc:milestone`.
