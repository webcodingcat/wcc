# Backlog

BACKLOG.md is a parking lot: work worth a plan that no phase asked for.
Nothing in it is scheduled automatically. The user sees how many items wait,
grouped by severity, at the start of every phase and in `/wcc:status`, and
takes items into work with `/wcc:change`.

## Taking items in

Reviewers, the verifier, agents and `/wcc:feedback` report candidates. Write
one to BACKLOG.md only if it states what goes wrong for whom and has one of
these severities:

- `security`: a security problem
- `data`: data can be lost or corrupted
- `bug`: a likely bug in normal operation, outside the requirements of the
  plan
- `later`: something a named later phase will need
- `minor`: a cosmetic finding the user reported and wants to keep (only
  from `/wcc:feedback`)

Drop the rest without writing it down: cases that need conditions the
software never creates, internal cleanups, matters of taste, and missing
tests for behaviour the requirements do not promise (a requirement without
its test is a blocker, not backlog). Most reviews add no item. A finding
that needs a change to the requirements or the architecture is an open
decision, not backlog.

Format, newest last:

```
- [ ] <severity>: <component>: <what goes wrong for whom> (source: <phase> <plan> review)
```

## The hint

Count the open items per severity and show one line, most severe first, for
example `Backlog: 1 security, 2 bug, 3 later - /wcc:change takes items in`.
Show nothing if the backlog is empty.

## Backlog phases

`/wcc:change` turns the items the user chose into a phase `- [ ] B<n>
Backlog: <short title>` in ROADMAP.md, with the items in
`phases/B<n>/PLAN.md` under "Items". `/wcc:phase` picks it like any other
phase:

1. Group the items by component into plans of a few items each, within the
   size limits of a plan. Documentation gaps go into the documentation
   plan. Items about the tests follow "Findings about the tests" in
   `feedback.md`.
2. An item gets a row in `TESTS.md` only if it changes behaviour a
   requirement promises or fixes a bug a test can show. Hardening, cleanup
   or documentation needs no new test; the existing tests stay green.
3. Its own reviews add only `security`, `data` and `bug` items.
4. Before the merge, check off the worked items in BACKLOG.md and move every
   checked item, done or dropped, unchanged with its notes to
   `BACKLOG-DONE.md` under a heading `## B<n>`, newest first. Create the file
   from `../../idea/assets/BACKLOG-DONE.md` if it is missing.
5. Close it like any phase.
