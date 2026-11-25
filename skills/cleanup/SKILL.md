---
name: cleanup
description: Clean up a wcc project after a finished milestone - remove the folders of its phases, which its tag keeps, point the references to them at the tag, and remove merged phase branches, leftover worktrees and stray files - so the next milestone starts from a lean repository, or the finished project is left tidy. Use this whenever the user says "clean up the project", "tidy the repository", "remove the old phases" or "clean up after the milestone", or when /wcc:status recommends it.
argument-hint: "[--check]"
---

# cleanup: tidy the repository after a milestone

Arguments: $ARGUMENTS

All planning documents live in `.project/`; file names below are relative
to it.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in;
this skill does not set one. A call without text carries no language: then
use the configured one, or the language of the conversation so far, never
the language of the documents or of this skill, which are English.

The folders of finished phases are read by no skill, and their plans and
assumptions are out of date; an agent searching `.project/` for a component
finds them next to what holds now. The milestone tag keeps every file, so
they can go once the milestone is tagged. What still holds lives in
DECISIONS.md, ARCHITECTURE.md, `research/EXPERIMENTS.md`, `decisions/`,
ROADMAP-DONE.md and the documentation; this skill leaves those as they are,
apart from the references below.

With `--check`, only report what would change, and write nothing.

## 1. Preconditions

Stop and say why if one does not hold:

- `.project/STATUS.md` exists and its first line is `STATUS: DONE`. Any
  other value means a phase or a manual check is still open; name the
  command `/wcc:status` gives instead.
- The project is a git repository, the base branch named in STATUS.md is
  checked out, and the working tree has no uncommitted changes to tracked
  files. If it has, show them and stop.
- Every milestone in ROADMAP.md whose phases are all checked has its tag
  `wcc-milestone-<n>` (`git tag --list`). A missing tag is set by
  `/wcc:phase` at its close; do not set it here.

## 2. Collect

Find, without changing anything:

1. **Phase folders**: each folder in `phases/` whose phase is a checked
   line in the section of a tagged milestone in ROADMAP.md, and which the
   tag contains (`git cat-file -e wcc-milestone-<n>:.project/phases/<id>`).
   A folder the tag lacks, or that "Open manual checks" in STATUS.md names,
   stays; name it.
2. **References**: every place in tracked files outside those folders
   that names a path inside them (`git grep -n "phases/<id>/"` per folder),
   in the planning documents, the project CLAUDE.md, the documentation
   and code comments.
3. **Branches**: local branches `wcc/*` that are merged into the base branch
   (`git branch --merged`). Unmerged ones are named and kept.
4. **Worktrees**: `git worktree list` entries other than the main one.
5. **Scratch**: `research/experiments/`, if it exists.
6. **Stray files**: untracked files that `.gitignore` does not cover
   (`git status --porcelain`), each with its path and size. Look into
   each one before naming it, so the user can decide.

## 3. Ask once

Show the result as one short list, grouped as above, with numbers (folders,
references, branches) instead of every path where there are many; name
every stray file. Ask once whether to apply it, and for each stray file
whether to delete it; a file the user wants to keep stays untracked. With
`--check`, stop after the list.

## 4. Apply

1. Rewrite each reference so it still leads to the file: a path
   `phases/<id>/<file>` becomes ``phases/<id>/<file> (tag
   `wcc-milestone-<n>`)``; the arrow `→ phases/<id>/` of a phase line in
   ROADMAP.md or ROADMAP-DONE.md becomes `→ wcc-milestone-<n>`. Change
   nothing else in those lines.
2. Remove the phase folders with `git rm -r -q`, one per command or in
   one command with all their paths.
3. Remove each worktree with `git worktree remove <path>`, never with
   `--force`. A worktree git refuses because of changes is named and kept.
   Then `git worktree prune`.
4. Delete the merged branches with `git branch -d`, never `-D`.
5. Delete `research/experiments/` and the stray files the user agreed to.
6. Commit the removed folders and the changed files, staged by name, with
   the message `Clean up after milestone <n>`. Do not push.

## 5. Report

At most five lines: what was removed, how many references point to the
tag now, what was kept and why. The status stays `DONE`. Next comes
`/wcc:change` for the next milestone; if the project is finished, there is
nothing more to do.
