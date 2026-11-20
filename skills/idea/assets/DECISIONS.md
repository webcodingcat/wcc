# DECISIONS

Every decision Claude takes without asking, and every assumption, is recorded
here, as long as it holds. Newest entries first.

Format:

## <date> <short title>

- Type: decision | assumption | change | approval
- Phase: idea | spec | phase <id> | change | feedback | upgrade
- What: <one sentence>
- Why: <one sentence>
- Cost of change: low | medium | high
- Replaces: <date> <title of the earlier entry>   (only if it replaces one)

The cost is high when a later change needs a migration or breaks something
others already use: data schema, file and save formats, public URLs and
interfaces, authentication. Claude never takes such decisions alone. They are
taken with the approval of the spec or by an answer from the user.

An entry that revokes or changes an earlier one states in full what holds
now, names the earlier entry under `Replaces:`, and the earlier entry moves
unchanged to DECISIONS-DONE.md, so this file holds only what is valid.

Keep entries short and their titles telling: phases find the entries
they need by their titles and do not read this file in full. Details belong
in ARCHITECTURE.md or the plan.
