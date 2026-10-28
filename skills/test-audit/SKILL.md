---
name: test-audit
description: Audit existing tests for meaningful failure detection, missing risk coverage, and reliability; report evidence and prioritized improvements without editing the suite.
---

# Audit tests

First read [test-principles](../test-principles/SKILL.md) before evaluating tests.
Read the [paired examples](../test-principles/references/examples.md) when they
help distinguish useful behavior coverage from implementation-shaped checks.

## Scope

Audit the feature, change, suite, or paths the user requested. Follow relevant
repository instructions. Inspect supporting code as needed to understand those
behaviors; do not expand into an unrelated repository-wide review.

This workflow is read-only: leave source, tests, fixtures, dependencies, and
configuration unchanged. Normal runtime test output and disposable test artifacts
are acceptable. Do not update snapshots, regenerate baselines, delete tests, or
apply fixes. Route requested follow-up planning to `test-plan`, implementation to
`test-write`, and diagnosis of flaky tests to `test-stabilize`.

## Establish the evidence

1. Identify the requested behaviors, users, contracts, and costly failure modes.
   Inspect the relevant implementation and nearby tests, not just test filenames.
2. Find the actual test commands, environments, fixtures, and external boundaries.
   Record which components run for real and which are replaced by test doubles.
3. Map the important behaviors to existing assertions. Include negative outcomes
   and durable effects where the behavior requires them. An assertion about a
   collaborator call may protect a contract, but may also miss the actual outcome.
4. For each suspected weakness, name a concrete, plausible broken behavior that
   would escape the test: for example, another user's records returned by the
   real route despite a passing test that replaces the authorization decision.
5. Check whether another relevant test already catches that behavior. Avoid
   declaring coverage absent after only a name search or an incomplete inspection.

## Evaluate protection and cost

- Prefer evidence that real components work together at a useful boundary.
  Recommend E2E coverage for critical journeys when it adds wiring or UI evidence;
  use integration coverage for detailed service, persistence, and failure behavior.
- Retain valuable unit coverage of rules, algorithms, and edge cases. The question
  is what failures a test catches, not whether its filename says "unit".
- Look for mocks that bypass the decision under test, assertions that repeat the
  fixture, false success paths, leaked state, arbitrary sleeps, and unverified
  background work. Explain their effect on a specific behavior before reporting.
- Treat duplicate coverage, snapshots, and interaction assertions contextually.
  Recommend consolidation only when the lost protection and retained coverage are
  understood. Never recommend removal based on test count or file type alone.
- When useful, run existing documented tests in their intended local environment.
  Inspect the command before running it; avoid update or external mutation modes.
  Do not install dependencies or repair the harness as part of this audit.
- Report executed commands, results, and execution limits. A passing suite proves
  only what was exercised. An environment failure does not establish a product bug.

## Deliver the audit

Lead with the most consequential findings. For each actionable finding include:

- **Evidence:** file and test references, relevant assertion/setup, and any run result.
- **Risk:** the concrete broken behavior and its user or system consequence.
- **Classification:** uncovered risk, weak/redundant test, reliability issue, or
  demonstrated product defect. Missing coverage alone is not a proven defect.
- **Next step:** a specific behavior to verify, test boundary, and observable result;
  for removal proposals, identify the test that preserves the useful protection.
- **Priority and confidence:** explain the practical urgency and evidence limits.

Summarize meaningful existing strengths and remaining uncertainty. Distinguish
observed facts from hypotheses, and inspected paths from uninspected scope.
Use no numeric quality score, coverage quota, or desired test-count target.
If there are no actionable findings, say so without inventing work.
