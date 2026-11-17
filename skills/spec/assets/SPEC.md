# SPEC: <project name>

Goals and non-goals are in IDEA.md and are only referenced here.

## Stack

- <language, framework, tools, each with its version> (reasoning and sources
  in DECISIONS.md)

## Accounts and secrets

What the user has to provide before the implementation runs. Values never
appear here, only names.

- <service or account>: <name of the environment variable>, <what for>
- none

## Machine prerequisites

- Elevated rights (administrator, root): <none | what for: build, tests or
  operation, and why they cannot be avoided>
- <further prerequisites, e.g. virtualization enabled, free ports>

## Requirements of the first version

### R1 <short title> (goal <n> from IDEA.md)

<Description in one to three sentences.>

Acceptance criteria:

- R1.1 <checkable statement> [auto | browser | manual]
- R1.2 <checkable statement> [auto | browser | manual]

## Documentation

The documents the project delivers, from "Documentation" in IDEA.md. Every
milestone brings them up to date for what it delivers: the last task of each
milestone plan is the documentation task, carried out by the `doc-writer`
agent. They describe what is built, never the target vision.

- D1 User documentation: <path, e.g. README.md or docs/user/>, for
  <audience>, <form: guide, reference, help inside the program>, covers
  <what>
- D2 Developer documentation: <path, e.g. README.md or docs/dev/>, for
  <audience>, covers setup, tests, architecture overview, conventions,
  extension points
- <or: none, with the reason from IDEA.md>

Acceptance criteria. They hold at the close of every milestone, measured
against what is built so far, and are not assigned to one milestone:

- D1.1 <e.g. every command the program lists in its help is described in
  docs/user/commands.md> [auto | manual]
- D1.2 <e.g. every code example in the user documentation runs> [auto]

## Out of scope

<What the first version deliberately leaves out, with a reference to the
target vision or the non-goals in IDEA.md.>
