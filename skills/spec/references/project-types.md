# Architecture and verification rules per project type

## API / infrastructure tool

- One shared core (API layer, state store, driver for the real environment)
  and one module per resource type, all with the same interface. A new
  resource type is a new module, never a change to the core.
- If compatibility with an existing API is a goal, the real client tools are
  part of the test harness.
- Requirements are almost entirely `unit`; `e2e` only where the real
  environment or the real client tools are the point, and behaviour all
  resource types share is one requirement, not one per type.
- Documentation: the reference is generated from the API description
  (for example OpenAPI) where the stack allows it, or checked against it by
  a test: every endpoint or resource type appears in the reference.

## Web app

- Requirements about the user interface are verified with a
  browser-driven test tool (`e2e`), the logic behind them with `unit`.
- Write UI-SPEC.md. Look and feel cannot be tested, so it is fixed in
  writing instead. If the user accepted the offer in the interview, plan one
  or two screen drafts for approval before phase 01 and update UI-SPEC.md
  from the result.
- Keep business logic out of the UI layer so it can be tested without a browser.
- Documentation: the user guide uses the names of views and buttons from
  UI-SPEC.md, so a test can check that every view it mentions exists.
  Screenshots age with every phase; use them only where text cannot
  explain a view.

## Game

- Game logic is separated from rendering and input. The logic runs without
  graphics and is deterministic for a given seed, so it can be tested like any
  other code.
- Write UI-SPEC.md for menus, HUD and visual style.
- All tuning values (speeds, costs, durations) live in configuration files, so
  the user can adjust them after a play session without a phase.
- Every phase ends with a play session by the user (`manual`) and is
  marked `Stops for: manual check` in the roadmap. Expect IDEA.md to change
  afterwards more often than for other project types.
- Documentation: controls and rules are read from the same configuration
  as the game where possible (key bindings, tuning values), so they cannot
  drift apart.

## CLI / library

- The public interface (commands, flags, function signatures) is specified
  with examples in ARCHITECTURE.md and tested through that interface.
- Documentation: the examples in the README and the reference run as tests
  (doctests, or a test that executes them), and a test checks that every
  command in `--help` is described.
