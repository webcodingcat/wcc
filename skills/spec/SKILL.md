---
name: spec
description: Derive REQUIREMENTS.md, SPEC.md, ARCHITECTURE.md and ROADMAP.md from IDEA.md, choose the tech stack, check feasibility and present a short summary for approval. Use this whenever IDEA.md exists and the user wants the spec, the stack, the architecture or the plan for a project, or says "continue with the spec".
---

# spec: derive the requirements, the spec, the architecture and the roadmap

REQUIREMENTS.md, SPEC.md, ARCHITECTURE.md and ROADMAP.md are Claude's documents. The user reads
only the summary at the end, so these documents must be correct without them
reading the details.

All planning documents live in `.project/`; file names below are relative to it.

## Preconditions

- STATUS.md says `IDEA_DONE`: start with step 1.
- STATUS.md says `SPEC_REVIEW`: an earlier run stopped while waiting for
  approval. Do not derive anything again; go straight to step 7 and show the
  summary of the existing documents.
- Anything earlier: point to `/wcc:idea` and stop. Anything later: the spec is
  approved; changes go through `/wcc:change`.
- No clarification session is still open.

## Language

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in,
also for the summary in step 7; this skill does not set one. A call without
text carries no language: then use the configured one, or the language of
the conversation so far, never the language of the documents or of this
skill, which are English. Results of agents and messages from other
sessions arrive in English, like messages; they do not change the user's
language. The options quoted below are given in English:
offer them in the user's language. Everything written to the repository is
English: REQUIREMENTS, SPEC, ARCHITECTURE, ROADMAP and the other planning documents, code
and repository docs. Other people may work on the project.

## Step 1: stack

Read everything in `decisions/`. A decision recorded there, for example from a
stack discussion during the interview, is binding. Respect the constraints in
IDEA.md. Otherwise choose the stack yourself. Weigh, in this order: fits the
problem, the user can maintain it, simplicity, mature tooling for automated
tests.

Do not choose from memory. Before deciding, give the candidates to the
`researcher` agent: current stable versions, whether they are maintained,
whether the parts work together in these versions, and known pitfalls for
this use. A stack planned on outdated knowledge fails late and expensively.
Write the versions into SPEC.md and record the choice, the rejected
alternatives and the sources in DECISIONS.md.

## Step 2: REQUIREMENTS.md and SPEC.md

Use `assets/REQUIREMENTS.md` and `assets/SPEC.md`. Rules:

- A requirement is one checkable capability in one line, with the number of
  its goal in IDEA.md: "user can reset the password with a link from an
  email", not a feature with a list of details. A feature with several
  promises becomes several requirements. IDs are a category prefix and a
  number (`AUTH-01`), grouped by category, never changed or reused.
- Every requirement states its test level, chosen by what it promises:
  - `e2e`: something about the real environment or the user's browser (a
    machine boots, a firewall blocks, a page shows a result). Costs minutes
    per run.
  - `unit`: logic, validation, mapping, formats. Costs seconds.
  - `none`: wiring, registration, configuration or data types that the
    tests of the behaviour around them cover. No test of its own.
  - `manual`: only what no test can judge (a look, a feel); the user checks
    it, guided by `/wcc:report`.
  Choose the cheapest level that proves the promise; the user confirms the
  levels with the summary (step 7).
- Detail the first milestone. What the target vision will need goes under
  "Later" without a level; what is excluded under "Out of scope" with the
  reason.
- List under "References" every external description the project must
  match (an API it re-implements, a file format, a protocol), with where
  its copy is read and its version. Planning and research read the part
  of a phase there; field names, codes and limits from memory are often
  wrong.
- List under "Accounts and secrets" every account and credential the
  project needs, each with where it comes from: an environment variable by
  name, or the existing login of a tool the user already uses (the active
  or a named context of a cloud CLI, `gh auth`), which needs no second copy
  of the secret. Such a credential is used by letting the tool make the
  call, also to another endpoint of the same provider (`hcloud --endpoint
  <url> api <path>`, `gh api`); it is read out of the tool only for a
  program that cannot use the tool, and only inside that one command. `/wcc:prepare` checks them while the user is present, so a
  missing key does not stop the run. This includes the access agents need
  to run examples or checks against a running instance of the project (a
  token for a local development service): agents never create accounts or
  credentials themselves. If the project itself issues that access, mark
  the entry optional until the phase that builds it.
- Name the project's identifiers in code under "Stack": module or package
  name, repository URL if known. The scaffold, imports and every phase use
  them, so a later change touches every file.
- Fill "Release" when IDEA.md or the stack says how the project is
  published (a package registry, release artifacts, a deployment);
  otherwise write `none`. Building what it needs (a CI workflow, a
  release workflow) is a phase like any other.
- Derive "Documentation" from IDEA.md and `references/project-types.md`:
  one entry `D<n>` per document with path, audience and content, kept apart
  from `.project/`, which holds planning documents and is no documentation
  for anyone but Claude. Its requirements go into category `DOC` of
  REQUIREMENTS.md: what a test can check where possible (every command,
  endpoint or setting is described; every example runs; every link
  resolves), `manual` only for readability, phrased against what is built,
  so they hold at the close of every phase. If a documentation tool is needed
  (a site generator, a reference generator), it is part of the stack. If
  IDEA.md says "none" for both, write "none" and the reason.
- State under "Machine prerequisites" whether build, tests or
  operation need administrator or root rights, and for what. Avoid the need
  where the design allows it: a run with elevated rights gives every agent
  those rights. If it cannot be avoided, the user has to start Claude Code
  elevated for the phases, and has to know that before approving.

## Step 3: ARCHITECTURE.md

Use `assets/ARCHITECTURE.md` and read `references/project-types.md` for the
rules of the project type. Rules:

- Design interfaces and structure for the whole target vision from IDEA.md.
- For every item of the target vision write one or two sentences on how it will be
  added later without changing the core. If the honest answer is "the core has
  to be rebuilt", the design is wrong: fix it now.
- Do not rule out anything listed under "Possible later".
- Describe the interfaces between components precisely enough that components
  can be built in parallel.
- Settle every decision with high consequences now: data schema, file and
  save formats, public URLs and API shapes, authentication, anything that
  later needs a migration or breaks clients. During a phase nobody is there
  to take them, and a phase that meets an open one has to wait for the user.
  Record each in DECISIONS.md with `Cost of change: high`.

If the project has a user interface, also write UI-SPEC.md from
`assets/UI-SPEC.md`. Agents that build screens in parallel each invent their
own look unless the decisions are fixed beforehand. Keep it as short as the
project's interface is small; decide what the user did not specify and record
it as an assumption.

## Step 4: ROADMAP.md

Use `assets/ROADMAP.md`. A milestone is a version of the project; a phase is
a slice of it that runs and can be checked. Rules:

- Plan milestone 1 in phases. List the target vision only roughly under
  "Later".
- Every phase delivers something that runs and can be checked, names the
  requirements it covers, and has two to five success criteria: results
  the user can observe when it is done ("a new user can sign up and log in"),
  not tasks. They guide the planning, the verifier and the manual checks.
- Phase 01 is a walking skeleton including the test harness against the
  real environment (real hypervisor, real browser, real database). The bare
  project scaffold is not part of it; `/wcc:prepare` creates that.
- Every requirement of milestone 1 is covered by exactly one phase; fill
  "Coverage" in REQUIREMENTS.md.
- Name what a phase depends on. Mark every phase whose `manual`
  requirements later phases depend on with `Stops for: manual check`.

## Step 5: feasibility

Run the `feasibility-checker` agent on IDEA, SPEC and ARCHITECTURE. It finds
the riskiest assumptions and verifies them by research or a small experiment
in `research/experiments/`. If an assumption fails, change the architecture now and
record it in DECISIONS.md.

The agent installs nothing. If it reports missing tools, show the user the
list in one go: tool, version, the check that waits for it, install command,
and whether the project needs it anyway (`/wcc:prepare` would ask for it
later) or only this check (it can be removed afterwards). Offer
"Installed" and "Skip". After "Installed", check that the tools
are really there and run the agent again for the checks that were waiting.
A tool the project itself needs goes into the stack or into
"Machine prerequisites" in SPEC.md, so `/wcc:prepare` checks it on every
machine.

The agent cannot get administrator rights. For every check it returns as
`needs elevated rights`, the user is present, so ask now instead of
guessing: say what is being checked and why it matters, give the exact
command to run in a terminal with elevated rights, and offer "Report
result" and "Skip". Evaluate a reported result like any other. A
skipped check, for a missing tool or for missing rights, becomes an entry in
DECISIONS.md with `Type: assumption` and `Cost of change: high`, and appears
in the summary.

If the agent reports that the project itself needs elevated rights, write
that into "Machine prerequisites" in SPEC.md.

When no check waits for the user any more, write `research/EXPERIMENTS.md` from
`assets/EXPERIMENTS.md`: one entry per checked assumption, with what is
needed to repeat the check. Refer to the entry (`E<n>`) in DECISIONS.md
wherever a decision rests on it. Then delete `research/experiments/`. The
code of an experiment is not reused, only what it showed; keeping the
summary instead of the code leaves nothing that ages or gets built by
accident.

## Step 6: review

Run the `spec-reviewer` agent, once. Fix blockers. Fix its notes now if
cheap, otherwise drop them; the backlog is for the code.

## Step 7: summary and approval

Set STATUS.md to `SPEC_REVIEW`. Then give the user a summary of at most ten
lines:

- stack, with one line of reasoning
- components
- documentation: which documents, for whom, or none
- the phases of milestone 1
- test levels in the user's terms, for example "38 requirements are checked
  in code (seconds), 12 against the real environment (about 15 minutes per
  full run), 6 get no test of their own, 4 you check by hand", and after
  which phases the run stops for a manual check. List the `e2e`, `none` and
  `manual` requirements one line each below the summary: there a wrong level
  costs time or safety. The user may change any; "I don't know" keeps your
  choice, recorded in DECISIONS.md as `Type: assumption` with `Cost of
  change: low`.
- what the user has to provide before the run: accounts, keys, tools, and
  elevated rights if the project needs them
- the decisions with high consequences from DECISIONS.md, and the most
  consequential assumptions, together at most five. If there are more
  decisions with high consequences, show them all in a list below the
  summary: the user approves them here, and nowhere else.

The user is present here, so you may ask bundled questions in the same format
as in the interview. Wait for approval.

If the user changes something (for example a different stack or an additional
tool), work it into all affected documents and record it in DECISIONS.md. If the
change touches the idea itself, change IDEA.md first.

After approval set STATUS.md to `READY` and tell the user the next step is
`/wcc:prepare`, followed by `/wcc:phase`.
