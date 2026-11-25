---
name: idea
description: Turn a raw project idea into IDEA.md through a structured interview. Use this whenever the user says "I have an idea", wants to start a new project, or wants to capture goals and non-goals for a project, even if they never mention IDEA.md.
argument-hint: "<description of the idea>"
---

# idea: interview the user and write IDEA.md

The idea: $ARGUMENTS

IDEA.md is the user's document. It holds the what and the why, never the how.
It is the only source for goals and non-goals; later documents refer to it.

All planning documents live in `.project/`; file names below are relative to
it. Only the project CLAUDE.md sits in the project root, because Claude Code
loads it from there.

## Language

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in;
this skill does not set one. A call without text carries no language: then
use the configured one, or the language of the conversation so far, never
the language of the documents or of this skill, which are English.
Results of agents and messages from other sessions arrive in English, like
messages; they do not change the user's language. The
options and phrases quoted below are given in English: offer them in the
user's language.

Everything written to the repository is English: IDEA.md and all other
planning documents, code, code comments and repository docs (README etc.).
Other people may work on the project, so the documents must not depend on the
language of the interview. If the user speaks another language, translate
what they said; keep the meaning, not the wording.

## Step 0: scaffold the project

If these files do not exist yet, create them from `assets/` in this skill folder:

- `CLAUDE.md` in the project root from `assets/project-CLAUDE.md`. If the
  project already has a CLAUDE.md, append the template content under a heading
  `## wcc rules`. Set the line `wcc version:` to `version` from
  `.claude-plugin/plugin.json`, two folders above this skill folder;
  `/wcc:upgrade` compares it with the installed plugin later.
- `.project/DECISIONS.md`, `.project/BACKLOG.md`, `.project/STATUS.md` from
  the matching templates.
- an empty `.project/decisions/` folder.

Set STATUS.md to `STATUS: IDEA`.

If `decisions/prototype-learnings.md` exists, read it first and do not ask again
what it already answers.

## Step 1: interview

The user is present during the interview, so asking is cheap. Ask instead of
assuming. There is no cap on the number of questions; the limit is on their kind.

Rules:

1. Ask only what the user can answer: problem, goals, non-goals, growth,
   constraints, success criterion, project type, who needs documentation,
   where the code lives and how it is published.
   Decide technical details
   yourself later.
2. Every question must be answerable in seconds: offer 2 to 4 options and mark
   one as your recommendation. Use the AskUserQuestion tool if it is
   available; otherwise, for example in a remote session where its menus do
   not show, ask in text with numbered options the user answers by number.
   Bundle questions into rounds of at most three.
3. Every question also offers "I don't know" and "I need more details for
   this (separate session)". "I don't know" means: take the recommendation
   and record it as an assumption. "More details" triggers a clarification
   session (step 2).
4. Ask an extra question only if different answers would lead to a different
   scope or a different architecture.
5. Options the user did not pick are neither goals nor non-goals. List them
   under "Possible later"; the architecture must not rule them out.

Mandatory topics, each of which must end up answered or marked as an assumption:

- Problem: what is missing or annoying today
- Goals
- Scope of the first version
- Target vision: where the project is supposed to grow. Ask this explicitly,
  once.
- Non-goals
- Constraints, including exactly one question about stack or tool
  requirements. If the user wants to discuss the stack, that discussion
  happens here as a clarification session, not later in the spec.
- Success criterion: how the user notices that the first version is done. It
  must be checkable.
- Documentation, once for end users and once for developers: who has to use
  or work on the result without asking, and in which form they get
  documentation (README, guide, reference, help inside the program, or none).
  Offer the recommendation for the project type from
  `references/project-types.md`. Ask this also for small projects: the
  answer decides whether every phase writes documentation, and
  documentation added later has to describe everything at once.
- Repository and publication: where the code lives (only on this machine,
  a private repository on a host such as GitHub or GitLab, or a public
  open-source project, then with its licence), and whether versions are
  published (release artifacts, a package registry, a deployment, or not
  at all). A host brings a CI from the first phase on, so problems a fresh
  machine shows come up early, not at the end of a milestone.
- Project type: read `references/project-types.md` and ask the additional
  mandatory questions for that type.
- Project name: ask in the last round, when the idea is clear. Ideas often
  start before they have a name, so the folder may carry only a codename.
  Offer two or three short names derived from the idea, each with the slug it
  would get as folder and package name, and mark one as your recommendation.
  If the folder name already looks like a deliberate name, offer it as the
  recommendation. Always offer "Keep a working title, decide later"; then
  the heading of IDEA.md becomes `# IDEA: <working title> (working title)`,
  which `/wcc:prepare` looks for. Do not check domains or package registries
  unless the user asks.

## Step 2: clarification sessions

A long discussion would fill the context of the interview, so it runs in a
child session. When the user answers "I need more details for this":

1. Save the current draft of IDEA.md to disk.
2. Note the question in the draft under "In clarification" with its result file
   `decisions/<topic>.md`, and continue the interview with the rest. The
   note on disk is what keeps the open question from being forgotten.
3. Start a child session with `mcp__ccd_session__start_session` (Claude Code
   Desktop; load it with ToolSearch if it is only listed as deferred). Use a
   fresh context and `use_worktree: false`, so the child works in this folder
   and can write its result here. The brief must be self-contained:
   - the question and why it matters for this project
   - the path to the IDEA.md draft
   - the instruction: talk to the user in their language; present the options with
     pros and cons for this specific project, answer follow-up questions,
     decide nothing without the user.
     Once the user has decided, and not before, write decision, reasons and
     rejected alternatives in English to `.project/decisions/<topic>.md`.
     Then tell the user in one line that the result is saved and that they
     can return to the interview session.
4. If the session does not start by itself but shows a card, ask the user to
   start it. If no session tool is available, print the brief and ask the user
   to open a new session with it.

The result file is the signal that a clarification is finished, not a message
from the child. A child session the user talks to does not reliably notify
this session, and a notification carries only the first few hundred
characters anyway. So do not wait for one:

- After every interview round and at the start of every turn, check whether
  the result file of a question "in clarification" exists. If it does, read it,
  update the draft and remove the note.
- If a notification does arrive, treat it only as a reason to check the file.
- Before step 3, every question "in clarification" must be resolved. For each one
  still without a file, ask the user: the discussion is finished (then the
  file is missing: ask for the decision here and record it), it is still
  running (stop here; the user calls `/wcc:idea` again afterwards), or drop
  it (take your recommendation and record it as an assumption).

When this skill is called and STATUS.md already says `IDEA`, an earlier
interview was interrupted: read the IDEA.md draft and `decisions/`, resolve
the questions "in clarification" as above and continue with what is still missing
instead of starting over.

Short questions of understanding that need no decision can be answered inline.

## Step 3: confirm assumptions

Before finishing, show the user the list of everything you assumed or would
decide yourself and that affects what they will experience:
"This is how I would decide. Object where needed." Apply corrections. Record the
confirmed assumptions in DECISIONS.md with `Type: assumption`.

## Step 4: write IDEA.md

Use `assets/IDEA.md` as the structure. Keep it to about one page.

The interview is complete only when all of these hold:

- every section of the template has an answer or a marked assumption
- no question is "in clarification" any more
- the success criterion is checkable

## Step 5: gap check

Run the `idea-gap-checker` agent on IDEA.md, once. It returns the points it had
to guess. Take the ones that affect scope or architecture back to the user as
questions (same format as in step 1) and update IDEA.md. Everything else is left
for the spec.

## Step 6: hand over

Set STATUS.md to `STATUS: IDEA_DONE`. Tell the user in two or three sentences
what IDEA.md contains and that the next step is `/wcc:spec`.

If the user chose a name and the folder name is not its slug, recommend
renaming the folder now. Until `/wcc:prepare` nothing depends on the path:
there is no git repository, no remote, no package name yet, and every
planning document refers to the project by the name in IDEA.md. Later a
rename drags real work with it. You cannot rename the folder yourself,
because this session runs inside it, so give the user the steps:

1. close this session
2. rename the folder to the slug
3. open a new session in the renamed folder and run `/wcc:spec`

Mention that the history of this session and of any clarification session
stays with the old path in Claude Code; the documents do not need it,
because `/wcc:spec` reads everything from `.project/`.
