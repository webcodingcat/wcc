# SPEC: <project name>

Goals and non-goals are in IDEA.md, the requirements in REQUIREMENTS.md.

## Stack

- <language, framework, tools, each with its version> (reasoning and sources
  in DECISIONS.md)
- Module or package name: <name>; repository: <URL or "not yet">

## References

External descriptions the project must match, such as an API it
re-implements, a file format or a protocol. Agents read the part a phase
touches here instead of relying on memory.

- <name>: <path of the copy in the repository or on the machine, or URL>,
  <version or pin>, <what it settles>
- none

## Accounts and secrets

What the user has to provide before the implementation runs. Values never
appear here, only names.

- <service or account>: <name of the environment variable, or "login of
  <tool>" with the context it uses>, <what for>
- none

## Machine prerequisites

- Elevated rights (administrator, root): <none | what for: build, tests or
  operation, and why they cannot be avoided>
- <further prerequisites, e.g. virtualization enabled, free ports>

## Documentation

The documents the project delivers, from "Documentation" in IDEA.md. Every
phase brings them up to date for what it delivers, in its documentation
plan, carried out by the `doc-writer` agent. They describe what is built,
never the target vision. Their requirements are in category `DOC` of
REQUIREMENTS.md.

- D1 User documentation: <path, e.g. README.md or docs/user/>, for
  <audience>, <form: guide, reference, help inside the program>, covers
  <what>
- D2 Developer documentation: <path, e.g. README.md or docs/dev/>, for
  <audience>, covers setup, tests, architecture overview, conventions,
  extension points
- <or: none, with the reason from IDEA.md>
