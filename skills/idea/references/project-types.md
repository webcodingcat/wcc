# Additional mandatory interview questions per project type

Ask the project type first: API/infrastructure tool, web app, game, CLI/library,
other. Then ask the questions of the matching section.

Each section ends with the recommendation for the documentation question.
For developer documentation the recommendation is the same everywhere: a
README with setup, tests and a short overview if only the user works on the
code; a separate developer guide (`docs/dev/`) as soon as others do.

## API / infrastructure tool

- Is there an existing API or tool it must be compatible with? Which clients
  have to work unchanged?
- On which real environment must it run and be tested (hypervisor, cloud, OS)?
- Documentation, recommended: a reference of every endpoint or resource
  with examples, plus a guide for installation and operation.

## Web app

- Who uses it: only the user, a small group, the public?
- Is a login needed, and against what (own accounts, SSO)?
- Look and feel: is there a reference the user likes? Offer to show one or two
  screen drafts before implementation starts.
- Documentation, recommended: none for users if only the user works with
  it; a short user guide per main workflow for a group or the public, plus
  an operation guide (installation, configuration, backup) if someone else
  runs it.

## Game

- What is the core loop, in one sentence: what does the player do again and again?
- Are the rules fixed (clone of a known game) or still to be discovered?
- If the rules are not fixed: recommend a throwaway prototype
  (`/wcc:prototype`) before the spec and ask whether the user wants one.
- Documentation, recommended: controls and rules, inside the game or in the
  README; more only if the game is published.

## CLI / library

- Who calls it: the user by hand, scripts, other programs?
- What does a typical call look like? Ask for one example.
- Documentation, recommended: `--help` for every command, a README with
  installation and typical calls, and for a library a reference of the
  public interface from doc comments.
