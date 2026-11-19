# End-to-end tests

End-to-end tests are the tests the project CLAUDE.md runs with a command of
their own, because they need the real environment or a browser and take
long. Every check against the real environment costs minutes, so they run
as rarely as their purpose allows.

## How they are written

The project CLAUDE.md carries these rules for the `test-writer`; the review
checks the tests against them.

- Expensive setup is shared. Tests that only use a running environment (a
  service, a VM, a database, a logged-in browser) share one instance for the
  whole run, brought up by the first test that needs it and torn down at the
  end. Each test isolates itself through its own data. A filter that
  selects one subtest still brings the setup up (in Go: in the parent test
  before `t.Run`, or in `TestMain`).
- Expensive steps are shared. Checks that need a restart, a recreation or a
  failover prepare their data before one shared run of that step and check
  afterwards, each as a subtest of its own. The lifecycle of the
  environment itself (start, stop, restart, teardown) is one sequence of
  subtests, not repeated in several groups.
- Booted machines are shared. Checks that only look at a server, VM or page
  (login, hostname, metadata, reachability) use one instance per variant.
- Independent subtests that mostly wait run in parallel (`t.Parallel()` in
  Go), within the capacity of the environment. Tests that change the shared
  environment never run in parallel with others. A test takes everything it
  needs from a shared limit (server slots, machines) in one step, never one
  part while it holds another; a shared setup that keeps a slot for later
  tests counts against the limit.
- The run shows its progress: each test is logged as it starts and ends
  (`-v` in Go), and the command's time limit is about twice its last full
  run, not hours. When a full run at a close ends far below the limit,
  lower the limit in the command to about twice that run.
- One subtest per row of `TESTS.md`, carrying its requirement ID and no
  other, under a group named after the component.
- No fixed waits: wait for a condition with a deadline. Time limits are
  checked in unit tests with short values or a test clock; an end-to-end
  test checks the real length at most once.

## What runs when

- During a plan: the `implementer` runs the plan's end-to-end tests once its
  fast checks are green, with the filter of its requirement IDs; after a
  repair only the ones that failed.
- At the close of a phase (also a feedback or backlog phase): only the
  subtests whose requirement the phase covers, and the subtests whose code
  it changed (from `git diff` over the phase, by test function, not by
  file). The unit suite and the linter run in full; they are fast.
- At the close of the last phase of a milestone, before its tag: all of
  them. That is the release check; during development only what a change
  can break runs.

Turn the selection into the filter of the framework (`-run` in Go, `-k` in
pytest, `--grep` in Playwright, `-t` in Jest), appended to the end-to-end
command. Match IDs exactly: `AUTH_01` must not select `AUTH_010`. Select
subtests, not whole groups. If nothing is selected, none runs. Never drop
the filter "to be safe", except after a change to the shared harness every
test depends on; then run all and say why. Note the exact command, or that
none ran, in `phases/<id>/PLAN.md` under "Close".

Do not run a test twice on the same code. A test that ran green during this
phase counts at the close if it ran on a commit after which `git diff
--name-only <commit> HEAD` shows no file the build or the tests use, and its
log still exists and shows this test itself as passed.

## After a fix

Run the unit suite and the linter in full again. For the end-to-end tests,
look at what the fix changed:

- product code, the shared harness, the build or the configuration: run the
  whole selection again.
- only single tests or helpers only they use: run the failed tests and the
  tests in the changed functions or using the changed helpers.
- a race or a flake: run the affected tests three times in a row.

A run that is also a measurement (a finding about run time) counts with its
duration even if single tests failed; say so instead of running everything
again.

A full run that fails at the end of a milestone found a regression: fix it
like a plan of the phase that is closing.
