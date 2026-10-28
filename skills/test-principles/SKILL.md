---
name: test-principles
description: Establish testing principles and decide what makes a test valuable when discussing test quality, test boundaries, or testing tradeoffs.
---

# Test principles

Use this foundation before auditing, planning, writing, or stabilizing tests with this collection. Apply it to the actual product and repository conventions; a test label does not establish its value.

## Start with a promise and a failure

State the observable promise in plain language: given a relevant state and action, what must happen, and what must never happen? Identify who or what depends on it. Find the requirement in a documented contract, accepted behavior, or user intent; do not assume current implementation is the specification.

Name a plausible defect the proposed test would catch. A test earns its cost by detecting a meaningful regression, clarifying a difficult contract, or making a risky change safer. Passing assertions, line coverage, and test counts alone do not demonstrate that value.

## Choose the boundary that contains the risk

Prefer real integration tests for behavior spanning application components. Include the actual routing, validation, authorization, serialization, transaction, persistence, or queue boundary responsible for the risk. Calling something an integration test does not make mocked components real.

Keep a small set of end-to-end tests for critical journeys and wiring that narrower tests cannot establish: an action through the real entrypoint, meaningful system behavior, and a visible or durable outcome. Use service or CLI entrypoints when there is no browser. Cover detailed permutations closer to their source unless the journey itself adds risk.

Use unit tests for dense business rules, parsers, algorithms, state machines, or expensive-to-reach edge cases with clear inputs and outputs. A unit test for rounding can be more valuable than an end-to-end test that only checks a page loads. Avoid trivial wrapper tests, private-method tests, and tests that reproduce the implementation with different syntax.

Choose the narrowest boundary that still contains the plausible failure. Explain a deviation from the integration/E2E preference when it affects confidence or cost. Do not replace useful existing tests just to change the suite's shape; do not add a test for every file or every reversible, low-impact edit.

## Use an independent oracle

Assert the observable outcome against a requirement or explicit fixture expectation, not a second call to the function under test. For state changes, inspect durable results or read through a separate public path. For rejection, verify the prohibited data or side effect is absent when that is the risk. An HTTP status or success toast alone may leave the important promise unchecked.

Multiple assertions are appropriate when they describe one coherent behavior. Use snapshots for intentionally reviewed stable contracts, not as a substitute for deciding what matters. Assert interactions when the interaction is the contract, such as issuing one external charge; prefer outcomes for internal collaborations.

## Keep real behavior real

Do not mock the behavior or boundary the test claims to cover. Use representative, isolated storage when database semantics matter; an in-memory replacement can miss constraints, transactions, dialect behavior, and concurrency. Local containers and disposable databases are options, not mandatory infrastructure for every test.

Replace external systems you do not control at their adapter or protocol boundary when needed for reliable execution. Exercise the actual application path, assert relevant outbound requests and downstream outcomes, and state what the replacement cannot prove. Use contract checks or sandbox integration checks for important provider assumptions when available and in scope. Never require live production services merely to make a test more realistic.

## Spend effort on consequential behavior

Prioritize critical journeys, permissions and isolation, money and data integrity, durable state transitions, boundary conditions, failure handling, retries, duplicate delivery, and recovery where relevant to the change. Derive cases from the failure modes of the product; this list is not a checklist to impose everywhere.

Usually skip framework internals, trivial accessors, static implementation details, and duplicate assertions with no new failure mode. Preserve exact strings, ordering, or markup when they are an actual interface, accessibility, localization, or rendering contract. Do not delete an existing test without understanding what unique guarantee would be lost.

## Make failures trustworthy

Isolate test-owned state and clean it up. Control time and randomness when they influence expectations. Wait for a specific observable condition with a bounded timeout instead of sleeping. Keep parallel workers independent. Failures should identify the broken promise and expose useful state, with credentials and sensitive data removed.

For a regression, demonstrate that the test fails for the intended reason before the fix and passes afterward when practical. A targeted, reversible fault injection can check an assertion's sensitivity when the original defect is unavailable. Do not mutate a live system or unrelated work to prove a test. Report what was actually run and any confidence limits; a passing test does not prove the absence of all defects.

## Calibrate with examples

Read the relevant pair in [references/examples.md](references/examples.md) when judging mock-heavy coverage, persistence, UI journeys, valuable unit tests, or flaky waits. The examples are explanatory sketches, not a framework to install or a suite to copy literally.

When using this skill alone, explain the applicable principles with a concrete failure and an appropriate test boundary. Do not launch a repository-wide audit unless requested.
