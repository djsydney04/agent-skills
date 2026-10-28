---
name: test-stabilize
description: Diagnose and fix a specific flaky or nondeterministic test while preserving the behavior it verifies. Use for intermittent failures, timing races, shared-state leaks, or unreliable test setup.
---

# Stabilize a failing test

First read [test-principles](../test-principles/SKILL.md). It defines the
collection's shared standard for test value, boundaries, and evidence.
Read the relevant [good and bad examples](../test-principles/references/examples.md)
when evaluating a proposed assertion, boundary, or synchronization change.

## Preserve the original evidence

Record the failing command, test identity, assertion or error, relevant logs and
artifacts, revision, and available environment details. Preserve a useful trace,
seed, or failing input before rerunning. Distinguish a test failure from a runner,
dependency, or infrastructure failure; each requires different evidence.

Establish the intended behavior and the assertion that protects it. A subsequent
passing run does not invalidate the original failure or establish a root cause.
Do not label a reproducible product defect as harmless test flakiness.

## Diagnose within a useful budget

Choose a small, explicit rerun budget appropriate to the test's cost. Start with
the failing test and the conditions implicated by the evidence: seed, order,
parallelism, environment, data, or resource pressure. Record outcomes rather than
rerunning until one passes. Preserve the original failing conditions as a baseline.

Inspect the path from setup through assertion and cleanup. Investigate supported
hypotheses, changing one relevant condition at a time where practical:

- A readiness race: an action finishes before its observable effect is available.
- Shared state: tests collide on records, ports, files, credentials, or global state.
- Incomplete cleanup: a failed test leaves resources or background work behind.
- Uncontrolled input: clock, timezone, randomness, locale, order, or external service.
- An ambiguous interaction: a selector, event handler, or request targets the wrong item.
- Resource or environment failure: capacity, dependency startup, or runner setup.

Connect the proposed cause to the captured failure with a trace, targeted
reproduction, controlled experiment, or direct code evidence. State confidence
and remaining uncertainty. If the budget is exhausted, report the evidence and
next discriminating experiment; do not invent a diagnosis.

## Fix the cause without removing the signal

Decide whether the defect is in the product, test, or harness. Fix product behavior
when that is the cause and within the authorized scope; otherwise report it with
the failing evidence. Preserve the test's ability to expose that product defect.

Use the smallest change supported by the diagnosis. For example, wait on the
observable state the next action requires, isolate test-owned records, reset
global state, or await and clean up background work. A readiness check needs a
bounded timeout and a useful failure message. Control clocks or random inputs
only where that matches the behavior the test is intended to exercise.

Do not suppress the failure with blanket retries, arbitrary sleeps, skipped tests,
swallowed exceptions, removed assertions, or wider timeouts without evidence.
Polling an eventually consistent contract can be appropriate when it observes
the required condition within a justified bound. Retrying the entire test is not
evidence that the underlying failure was fixed.

If an existing test is coupled to incidental internals, replace that coupling with
an assertion of the same behavioral contract. Explain how the new assertion still
catches the original defect; making the assertion easier to satisfy is not a fix.

## Verify under the relevant conditions

Run the focused test under the conditions that previously failed, then perform
bounded repeated runs justified by the suspected mechanism. Include neighboring
tests or parallel execution when shared state or ordering is implicated.
Run repository-required checks appropriate to the change.

Report the original failure, diagnosed cause and confidence, change, exact commands,
and observed pass/fail counts. Distinguish observed stability from a guarantee:
a finite run with no failures cannot prove that flakiness is eliminated. Identify
unreproduced failures, unavailable environments, and remaining product defects.
