# Migrations

One entry per plugin version whose change needs something in an existing
project. Every entry says how to detect that it is still needed, so it can be
checked on any project, whatever version it claims. Entries only for
versions that need nothing in a project are left out.

Changes to rules in the project CLAUDE.md need no entry here: `/wcc:upgrade`
compares the project CLAUDE.md with the current template anyway. An entry is
needed when a planning document needs a new section, a renamed key, content
in a new form, or a follow-up with another skill.

Format:

```
## <version> <short title>

- Detect: <what in the project shows that the entry is still needed>
- Do: <what /wcc:upgrade changes now>
- Follow-up: <skill the user runs afterwards, and why; omit if none>
```

Newest entries first.

## 0.32.13 `/wcc:feedback` is now `/wcc:report`

- Detect: a file in `.project/` or the project `CLAUDE.md` names
  `/wcc:feedback`.
- Do: replace `/wcc:feedback` with `/wcc:report` in those files. Leave
  every other "feedback" as it is (FEEDBACK.md, feedback phases, the
  `feedback/` image folders, `source: feedback`).

## 0.32.10 Auto mode entry for agent access

- Detect: "Accounts and secrets" in `.project/SPEC.md` names access agents
  use against a running instance through a tool's login, and
  `claude auto-mode config` lists no environment entry for that context.
- Do: show the user the `autoMode.environment` entry as step 3 of
  `skills/prepare/SKILL.md` describes it, for their own
  `~/.claude/settings.json`; do not write that file.

## 0.32.9 Permission rule for agent access

- Detect: "Accounts and secrets" in `.project/SPEC.md` names access agents
  use against a running instance through a tool's login (a named context
  of a CLI), and `permissions.allow` in `.claude/settings.json` has no
  rule for that tool and context.
- Do: add the rule as step 3 of `skills/prepare/SKILL.md` describes it,
  for that context only.

## 0.32.7 References in SPEC.md

- Detect: `.project/SPEC.md` has no heading `## References`.
- Do: add the section `## References` from `skills/spec/assets/SPEC.md`
  before "Accounts and secrets". Fill it with the external descriptions
  SPEC.md, ARCHITECTURE.md or DECISIONS.md already say the project must
  match (an API it re-implements, a file format, a protocol), each with
  the path of its copy and its version as those files state them; ask the
  user for a path no file states. Without any, write "none".
