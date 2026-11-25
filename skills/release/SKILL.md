---
name: release
description: Finish a wcc milestone - remove the folders of its phases, which its tag keeps, point the references to them at the tag, merge the milestone branch into main, tidy branches, worktrees and stray files, and, if SPEC.md describes a release, push and publish the version - so the next milestone starts from a lean repository, or the finished project is left tidy. Use this whenever the user says "release", "publish the milestone", "merge the milestone", "clean up the project", "tidy the repository" or "remove the old phases", or when /wcc:status recommends it.
argument-hint: "[--check] [version]"
---

# release: bring a finished milestone into main and publish it

Arguments: $ARGUMENTS

All planning documents live in `.project/`; file names below are relative
to it.

Talk to the user in the language they have configured (the `language`
setting or their own CLAUDE.md), otherwise in the language they write in;
this skill does not set one. A call without text carries no language: then
use the configured one, or the language of the conversation so far, never
the language of the documents or of this skill, which are English.

A milestone is built on its own branch `wcc/milestone-<n>`, the base branch
of its phases. `main` gets it only through this skill, after the phase
folders are gone: they are read by no skill, their plans and assumptions
are out of date, and the tag `wcc-milestone-<n>` keeps them. What still
holds lives in DECISIONS.md, ARCHITECTURE.md, `research/EXPERIMENTS.md`,
`decisions/`, ROADMAP-DONE.md and the documentation; this skill leaves those
as they are, apart from the references below. The milestone branch stays
for later fixes; after a feedback phase this skill runs again. A project
whose base branch is `main` itself skips the merge.

With `--check`, only report what would change, and write nothing. A
version given as argument is the version to publish.

## 1. Preconditions

Stop and say why if one does not hold:

- `.project/STATUS.md` exists and its first line is `STATUS: DONE`. Any
  other value means a phase or a manual check is still open; name the
  command `/wcc:status` gives instead.
- The project is a git repository, the base branch named in STATUS.md is
  checked out, and the working tree has no uncommitted changes to tracked
  files. If it has, show them and stop.
- The finished milestone has its tag `wcc-milestone-<n>` (`git tag
  --list`). A missing tag is set by `/wcc:phase` at its close; do not set
  it here.

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
3. **Branches**: local branches `wcc/phase-*` and plan branches that are
   merged into the base branch (`git branch --merged`). Unmerged ones are
   named and kept; milestone branches always stay.
4. **Worktrees**: `git worktree list` entries other than the main one.
5. **Merge**: whether the base branch is a milestone branch that `main`
   (or the default branch) does not contain yet.
6. **Scratch**: `research/experiments/`, if it exists.
7. **Stray files**: untracked files that `.gitignore` does not cover
   (`git status --porcelain`), each with its path and size. Look into
   each one before naming it, so the user can decide.
8. **Release**: the section "Release" in SPEC.md. If it says `none` or is
   missing, nothing is published. Otherwise check what it names under
   "Tools" and "Remote" (a login, the remote and its URL), and propose the
   version by its rule, or take the one given as argument. A version tag
   that exists already is never reused.

## 3. Ask once

Show the result as one short list, grouped as above, with numbers (folders,
references, branches) instead of every path where there are many; name
every stray file. For a release, name the version, the remote and each
step that leaves this machine (the push of `main`, the tag, what the tag
starts). Ask once whether to apply it, whether the version fits, and for
each stray file whether to delete it; a file the user wants to keep stays
untracked. With `--check`, stop after the list. If a tool or the remote
the release needs is missing, say how to set it up and leave the release
out of this run.

## 4. Tidy and merge

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
   the message `Clean up after milestone <n>`, on the base branch.
7. If the merge is due, check out `main` and merge the milestone branch
   with `git merge --no-ff wcc/milestone-<n>`, message `Milestone <n>`. On
   a conflict, abort the merge, name the files and stop.

## 5. Publish

Only with a release the user agreed to; otherwise leave the push to the
user. Follow the steps of the section "Release" in SPEC.md, from `main`:

1. Push `main` to the remote it names. Push nothing else: no milestone or
   phase branch, no `wcc-milestone-*` tag, never `--all` or `--tags`.
2. If it names a check before the tag (CI green on the pushed commit),
   wait for it inside your turn, with a time limit of about twice its
   usual run. If it fails, stop before the tag and report what failed.
3. Create the version tag as an annotated tag on the commit just pushed,
   and push it alone, by name.
4. Wait for what the tag starts, then run the check under "Done when".
   A tag that was pushed is never moved or deleted; if the release failed,
   report what is missing, and a repaired release takes the next version.
Check out the base branch again at the end. Note in STATUS.md under
"Current state" what reached `main` and the version published, and commit
it there.

## 6. Report

At most five lines: what was removed, how many references point to the
tag now, whether `main` holds the milestone, the version published and
where, what was kept and why. The status stays `DONE`. Next comes
`/wcc:change` for the next milestone; if the project is finished, there is
nothing more to do.
