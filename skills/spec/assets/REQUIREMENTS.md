# REQUIREMENTS: <project name>

One line per requirement: one checkable capability, its goal in IDEA.md, its
test level. IDs are a category prefix and a number; they never change and
are not reused.

Test levels: `unit` (checked in code), `e2e` (against the real environment
or a browser), `none` (wiring or configuration the tests around it cover),
`manual` (only what no test can judge; the user checks it).

## Milestone 1

### <Category, e.g. Authentication> (`AUTH`)

- AUTH-01 <user can ...> (goal <n>) [unit | e2e | none | manual]
- AUTH-02 <...> (goal <n>) [unit]

### Documentation (`DOC`)

They hold at the close of every phase, measured against what is built so
far, and belong to no single phase.

- DOC-01 <e.g. every command the program lists in its help is described
  in docs/user/commands.md> [unit]
- DOC-02 <e.g. every code example in the user documentation runs> [unit]

## Later

What the target vision in IDEA.md will need, not built yet. No test level
until a milestone takes it in.

- <CAT>-<NN> <...>

## Out of scope

| What | Why |
|------|-----|
| <feature> | <reason, with a reference to the non-goals in IDEA.md> |

## Coverage

| Requirement | Phase | Done |
|-------------|-------|------|
| AUTH-01 | 01 | |
