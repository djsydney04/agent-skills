# Evaluation prompts

Give the agent only the selected case and its supplied context. Use the actual skill directories alongside it. The examples below are TypeScript-style sketches; no runnable project or execution environment is supplied.

## A. Audit the protection

User request: "Audit the tests below. Give me the most useful improvements with evidence. Please don't edit anything."

Supplied context: the API contract says a tenant may list only its own invoices. The real `listInvoices` query is not included, and no other suite was supplied.

```ts
test('lists invoices', async () => {
  invoiceRepository.list.mockResolvedValue([{ id: 'i-1', tenantId: 'a' }]);
  const response = await handler({ user: { tenantId: 'a' } });
  expect(response.status).toBe(200);
  expect(response.body).toEqual([{ id: 'i-1', tenantId: 'a' }]);
});

test.each([[99, false], [100, true], [199, true], [200, false]])(
  'booking at %i is active: %s', (now, expected) => {
    expect(isBookingActive(now, 100, 200)).toBe(expected);
  },
);
```

The booking contract includes its start and excludes its end. `isBookingActive` is the real pure implementation. Neither test was run for this evaluation.

## B. Plan coverage for a backend retry

User request: "Plan tests for retrying invoice creation. We have a browser dashboard, but this endpoint is also used directly by API clients. Don't implement anything yet."

Supplied context: `POST /invoices` accepts an idempotency key scoped to the authenticated tenant. A replay with the same payload must return the same invoice and create one durable invoice; another tenant may use the same key independently. A conflicting payload must return 409. The API uses PostgreSQL. The existing browser suite already verifies that creating an invoice displays it after a reload. Status codes for first creation and successful replay are not supplied. Concurrent delivery is supported. No payment processor is involved.

## C. Implement a regression with incomplete runtime evidence

User request: "Use test-write to sketch a regression test for the bug where updating my shipping address overwrites another customer's address. Explain how you would verify it."

Supplied context: use the real authenticated HTTP update route and an isolated database. The contract is that a user can update their own address but receives 404 for another user's address ID; denial must leave both addresses unchanged. The bug fix is not supplied, and this evaluation has no executable harness.

## D. Investigate an intermittent import test

User request: "Use test-stabilize to tell me the next investigation and smallest justified fix for this intermittent test."

```ts
await api.startImport({ workspaceId: 'shared-workspace', file: 'three-rows.csv' });
await sleep(1000);
expect(await db.records.count({ workspaceId: 'shared-workspace' })).toBe(3);
```

Supplied context: in CI, the assertion sometimes sees 2 and sometimes 6 records. Tests run in parallel. Each import returns a job ID, and a read-only job endpoint reports queued, processing, completed, or failed. No trace or reproduction is supplied, so the precise cause is not established. Records expose the source job ID.

## E. Avoid unnecessary tests

User request: "Use test-plan to decide what tests to add for this change."

Supplied context: the entire change fixes a typo in a non-executable source comment. It changes no behavior, generated artifact, public documentation, or configuration. No repository rule requires a new test.

## F. Preserve a meaningful interaction assertion

User request: "Use test-audit to review this unit test. Should we remove it because it mocks a collaborator?"

Supplied context: after reading from a local test database, the real dispatcher must issue exactly one delivery request for an eligible message and none for a cancelled message. The queue client is the external adapter. Tests exercise the real dispatch decision with each state, substitute the queue client, assert the full outbound message and exact request count, and verify the saved delivery state. The tests do not cover the queue provider's live protocol. No duplicate-delivery or concurrency claim is made.
