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

No entries yet.
