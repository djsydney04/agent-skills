---
name: test-write
description: Write or improve focused automated tests for an identified behavior or regression. Use when implementing tests; use test-audit to assess a suite or test-plan to choose broader coverage.
---

# Write meaningful tests

First read [test-principles](../test-principles/SKILL.md). It defines the
collection's shared standard for test value, boundaries, and evidence.
Read the relevant [good and bad examples](../test-principles/references/examples.md)
when choosing an assertion, test level, or replacement for a weak test.

## Establish what the test must prove

- Identify the behavior, plausible regression, and consequence the test protects.
  Start from the user's requirement, documented contract, or observed failure;
  use the implementation to locate execution paths, not to invent the oracle.
- Inspect nearby tests, runner configuration, fixtures, and required checks.
  Reuse the project's existing tools and conventions when they serve the goal.
- Write down the observable outcome and the defect that would make it fail.
  If neither is concrete, resolve the contract before adding assertions.
- Keep the change focused on the requested behavior. Do not add unrelated tests,
  packages, test infrastructure, or product refactors to increase test counts.

## Choose a boundary that exercises the risk

Prefer an integration test through the real application boundary that can expose
the defect: request handling, persistence, permissions, serialization, or another
relevant collaboration. Exercise the components whose interaction matters.

Use a small number of end-to-end tests for critical user journeys where browser,
client, server, and storage behavior must work together. Avoid duplicating every
edge case through a slow journey when a closer boundary proves the same risk.

Use unit tests when isolated behavior has a meaningful contract, such as complex
calculation, parsing, or a state transition. Do not add a unit test merely because
a function exists, or mock its collaborators until the test only proves its setup.

Keep relevant internal boundaries real. Substitute an external system when real
access would be uncontrolled, unsafe, expensive, or impractical; place the double
at that boundary and state which compatibility claim remains unverified. Use a
contract or integration check when that compatibility is the risk being tested.

## Implement the proof

- Use a descriptive behavior name and minimal fixtures that expose the case.
  Keep essential preconditions visible; helpers should not hide what matters.
- Assert a meaningful result, persisted effect, denial, or absence of an unwanted
  effect. A successful status or visible screen alone may miss the actual defect.
- Derive expected results independently of the code under test. Avoid computing
  expected values with the same helper, algorithm, or serializer being checked.
- Check the behavior that matters without coupling to incidental call sequences,
  private methods, layout, or exhaustive snapshots. Assert calls when the call
  itself is the required effect, such as an external delivery request.
- Cover the relevant failure or boundary case as well as success when it protects
  a distinct risk. Choose examples by risk; avoid combinatorial filler.
- Isolate mutable data and clean up resources even after assertion failures.
  Control time and randomness when relevant. Wait for observable readiness with
  a bounded timeout instead of arbitrary sleeps or assumptions about execution order.

## Verify and report

Run the new or changed test with the project's normal command. For a regression,
verify that it fails for the intended reason against the defective behavior when
practical, then passes after the fix. A setup failure does not establish this.
If the defective revision cannot be run safely, explain the limit instead of
claiming a demonstrated regression check.

Run the related tests and repository-required checks appropriate to the change.
Investigate unexpected failures; do not weaken assertions to obtain a green run.
If validation is blocked, retain the focused change and report what was unverified.

Summarize the protected behavior, chosen boundary, commands and observed results,
and material gaps such as substituted services or an untested browser journey.
Keep passing test counts separate from evidence that the intended defect is caught.
