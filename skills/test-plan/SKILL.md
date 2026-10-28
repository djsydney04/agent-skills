---
name: test-plan
description: Plan a minimal, risk-based test suite for a feature or change, choosing useful test boundaries and observable assertions without implementing tests.
---

# Plan tests

First read [test-principles](../test-principles/SKILL.md) before evaluating tests
or choosing coverage. Consult the [paired examples](../test-principles/references/examples.md)
when deciding whether a proposed test would detect a real failure.

## Scope and inputs

Produce an actionable plan for the user's feature, change, or stated testing goal.
Inspect relevant contracts, implementation, existing tests, and repository guidance.
Use an existing audit when supplied; do not perform an unrelated broad audit.

This skill plans only: do not implement tests, change application code, install
dependencies, or redesign the test harness. Use `test-write` for implementation
and `test-stabilize` for investigation of flaky existing tests.

Resolve ordinary choices from the repository. If a missing behavior contract would
change the expected result, identify the ambiguity instead of inventing a rule.
Continue planning independent cases and label any assumptions that affect coverage.

## Select coverage by risk

1. State the observable promises and the users or systems depending on them.
2. Identify plausible failures and their consequences. Prioritize critical journeys,
   authorization boundaries, data integrity, money, irreversible effects, recovery,
   and complex rules when they are relevant to this task.
3. Map each risk to existing evidence. Reuse protection already present; do not
   prescribe another test merely because a changed file has no matching test file.
4. Choose the smallest practical test boundary that exposes the failure. Prefer
   meaningful integration tests with real application components and representative
   persistence for cross-component behavior. Use E2E tests for critical journeys
   whose UI, deployment, or system wiring requires that evidence.
5. Use unit tests for dense rules, algorithms, parsing, and boundary combinations
   when isolation makes failures clearer and feedback cheaper. Avoid trivial
   wrappers and internal call checks that add no distinct behavioral protection.
6. Define an oracle: the observable result that distinguishes correct behavior
   from the specific failure. Include forbidden side effects where relevant.
7. Remove proposed overlap unless a different boundary catches a distinct failure.
   Do not apply fixed test-level ratios, coverage quotas, or a test-count target.

## Make every proposed test implementable

For each case, specify:

- **Behavior and risk:** the promise and a concrete plausible violation.
- **Boundary:** entry point and components exercised; explain the chosen level.
- **Setup and stimulus:** meaningful data, actor, state, and action or failure.
- **Oracle:** response, visible result, durable state, or important absent effect.
- **Doubles:** any replaced boundary and what still verifies its real contract.
- **Reliability:** necessary clock control, isolation, cleanup, or bounded waiting.
- **Placement and priority:** likely existing suite/file and why this case matters.

Use concrete cases rather than headings such as "test edge cases" or "add E2E".
For a duplicate-submission risk, specify two requests sharing an idempotency key,
the real handler and database, and the expected single durable operation plus the
documented repeat response. Do not invent undocumented response semantics.

## Deliver the plan

Present a compact risk-to-boundary-to-oracle table in implementation priority order.
Identify existing coverage to retain and any prerequisites needed to run the cases.
Start with the smallest useful suite; separate optional expansion by its added value.

Explicitly state what not to test or duplicate and why: framework behavior already
trusted, trivial forwarding, incidental markup, unreachable states, or permutations
already covered at a cheaper boundary. These exclusions must fit the actual code.

Explain how completion will be checked using relevant existing commands and, for
regressions, failure before the fix when practical. Do not claim planned tests ran.
Close with unresolved contract questions or evidence limits that change the plan.
