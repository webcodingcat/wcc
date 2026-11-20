# Feedback

The user reports what they notice by hand with `/wcc:report`, which sorts
it with them and records it. While a phase is running, it writes to
FEEDBACK.md instead, so it does not edit files under the run's feet.

## FEEDBACK.md at the start

Before picking the phase, record its findings without questions, the user
has already confirmed them: exactly as step 5 "Record" of
`../../report/SKILL.md` says for each class, except that a wish goes to
"Open decisions" in STATUS.md with source `feedback`.

Commit the changed planning documents and delete FEEDBACK.md. If the run
resumes a phase on its branch, do this after that phase's merge.

## Feedback phases

A feedback phase fixes bugs the user found, and minor findings they did not
want to wait for, like any phase:

1. `phases/F<n>/PLAN.md` already holds the findings. Plan one plan per
   finding, or per group with the same cause. A finding that needs a change
   to the requirements or the architecture is a wish: mark its plan `[!]`
   with ` - wish, see /wcc:change`.
2. Every finding a test can show gets a row in `TESTS.md` with its
   requirement ID, and a test that fails now. A finding without a
   requirement is a bug of the requirement whose behaviour it breaks; if
   there is none, it is a wish. A finding no
   test can show (a look, a feel) gets none; at the close it goes to
   `CHECKS.md` so the user confirms the fix.
3. Implement, review and close like any phase, with the documentation plan
   if a fix changes what users or developers see. Delete the images in
   `phases/F<n>/feedback/` once it is closed. A milestone tag stays where it
   is.

## Findings about the tests

A finding can concern the tests rather than the product: the end-to-end run
takes too long, a test is flaky, the harness lacks something. The proof that
such a change is right is that the existing tests stay green with the same
coverage, plus a measurement where the finding is about run time or
flakiness.

- New logic in the harness (a limiter, the synchronisation of a shared
  step, a fixture with state) gets tests like any code.
- No tests that read the test sources or fix how tests are structured.
- Plan the change with its measurement: the command, the value before and
  where the result is noted (usually "Close" in the phase's `PLAN.md`).
  Where tests are restructured, `TESTS.md` lists for every requirement its
  test before and after, so the review sees that none was lost or weakened.
