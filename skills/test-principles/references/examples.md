# Weak and strong tests

These TypeScript-style sketches omit project-specific imports and fixtures. They are not runnable as a standalone suite. Names such as `app`, `db`, `seedUser`, and `providerStub` refer to the target project's test harness, not production services. Each pair holds the promise constant so the difference in confidence is visible.

## 1. Integration: prevent access to another tenant's document

Promise: an authenticated user cannot read another tenant's document, even if they know its ID. Assume the documented API hides the document with a 404.

Weak: replace authorization and persistence, then assert the handler followed the supplied answer.

```ts
canRead.mockReturnValue(false);
documents.find.mockResolvedValue({ id: 'doc-b', text: 'private' });
const response = await handler(requestFor('doc-b'));
expect(response.status).toBe(404);
```

This can check a denial branch, but does not establish tenant isolation. It can stay green when the real authorization policy or tenant query is broken.

Stronger: send a real authenticated request through routing, authorization, and the repository backed by an isolated database.

```ts
const alice = await seedUser({ tenant: 'a' });
const own = await seedDocument({ tenant: 'a', text: 'alice note' });
const other = await seedDocument({ tenant: 'b', text: 'private-b' });
const client = await app.clientFor(alice);

const allowed = await client.get(`/documents/${own.id}`);
expect(allowed.status).toBe(200);
expect(allowed.body.text).toBe('alice note');

const denied = await client.get(`/documents/${other.id}`);
expect(denied.status).toBe(404);
expect(JSON.stringify(denied.body)).not.toContain('private-b');
```

The positive control prevents an always-deny endpoint from passing. Removing the tenant predicate or allowing cross-tenant authorization should make the denial assertions fail. This test covers this document route, not every tenant boundary in the system.

## 2. Integration: duplicate delivery must not create two orders

Promise: sequential requests with the same idempotency key and payload return the same order and persist one order. Assume the documented API returns 201 initially and 200 on replay.

Weak: make one request and verify an internal collaborator was called once.

```ts
await createOrder({ sku: 'book', key: 'key-1' });
expect(repository.insert).toHaveBeenCalledTimes(1);
```

This says nothing about a second request, storage uniqueness, or persisted replay state.

Stronger: exercise the real HTTP handler and production-equivalent database semantics.

```ts
const customer = await seedCustomer();
const client = await app.clientFor(customer);
const payload = { sku: 'book', quantity: 1 };
const options = { headers: { 'Idempotency-Key': 'key-1' } };

const first = await client.post('/orders', payload, options);
const replay = await client.post('/orders', payload, options);

expect(first.status).toBe(201);
expect(replay.status).toBe(200);
expect(replay.body.id).toBe(first.body.id);
const stored = await db.orders.forCustomer(customer.id);
expect(stored).toHaveLength(1);
expect(stored[0]).toMatchObject({ sku: 'book', quantity: 1 });
```

Ignoring the key or storing replay state only in a request-local variable should fail this test. Concurrent submissions need a separate, coordinated concurrency test if race handling is part of the promise; two sequential calls do not prove atomicity. A changed payload using the same key also needs its own case when relevant to the API contract.

## 3. External boundary: a declined payment must not trigger fulfillment

Promise: a processor decline leaves the order in `payment_failed`, with no fulfillment job. Assume that is the product's documented state transition.

Weak: replace the checkout behavior being tested.

```ts
checkout.mockResolvedValue({ status: 'payment_failed' });
expect((await checkout(orderId)).status).toBe('payment_failed');
```

This checks the fake, and cannot detect an application defect.

Stronger: replace only the processor transport. Use real checkout logic, storage, and a test-owned queue backend whose enqueued messages can be inspected.

```ts
const order = await seedUnpaidOrder({ totalCents: 2500, currency: 'USD' });
providerStub.replyToCharge({ status: 402, code: 'card_declined' });

const response = await app.customer(order.customerId)
  .post(`/orders/${order.id}/pay`);

expect(response.status).toBe(402);
expect(providerStub.lastChargeRequest()).toMatchObject({
  amount: 2500, currency: 'USD', orderId: order.id,
});
expect((await db.orders.get(order.id)).status).toBe('payment_failed');
expect(await queue.fulfillment.forOrder(order.id)).toEqual([]);
```

Fulfilling before the charge succeeds, sending the wrong amount, or recording success on decline should fail. The queue inspection assumes enqueueing is awaited or the relevant test-owned worker has reached a known completion point. An immediate empty-queue check cannot prove that no delayed job will arrive. The processor stub does not prove the live provider accepts the request; keep important protocol assumptions aligned with a verified contract or sandbox check.

## 4. End-to-end: a note survives reload

Promise: a signed-in user can create a note through the UI and read its saved contents after reloading. The browser talks to the actual local application and an isolated database; no note API routes are intercepted.

Weak: intercept saving and check only immediate UI state.

```ts
await page.route('**/api/notes', route => route.fulfill({
  json: { id: 'fake-note', text: 'Buy milk' },
}));
await page.getByRole('textbox', { name: 'Note' }).fill('Buy milk');
await page.getByRole('button', { name: 'Save' }).click();
await expect(page.getByText('Saved')).toBeVisible();
```

This can be useful as a component-level UI test, but does not establish saving through the backend.

Stronger: use a test-owned account with no existing notes and reload the real saved view.

```ts
await signInAsTestUser(page, user);
await page.goto('/notes/new');
await page.getByRole('textbox', { name: 'Note' }).fill('Buy milk');
await page.getByRole('button', { name: 'Save' }).click();
await expect(page).toHaveURL(/\/notes\/[a-zA-Z0-9-]+$/);
const noteId = new URL(page.url()).pathname.split('/').pop()!;
expect(await db.notes.get(noteId)).toMatchObject({
  text: 'Buy milk', userId: user.id,
});
await page.reload();
await expect(page.getByRole('textbox', { name: 'Note' }))
  .toHaveValue('Buy milk');
```

Breaking the save route, its persistence, or loading the saved note should fail. The independent database assertion matters: reload alone can be satisfied by a client cache. This scenario establishes application-level saving and reading, not recovery after a database crash.

## 5. Unit tests: useful rules versus implementation echoes

Promise: a booking interval includes its start and excludes its end. This is a pure rule, so reaching it through a browser adds little value for boundary permutations.

Weak: compute the expected value by copying the expression under test.

```ts
const expected = now >= startsAt && now < endsAt;
expect(isBookingActive(now, startsAt, endsAt)).toBe(expected);
```

If both expressions share the same mistaken boundary, the test preserves the mistake. A wrapper test that only asserts `isBookingActive` was called is weaker still.

Stronger: derive explicit cases from the stated interval contract.

```ts
test.each([
  [99, false],
  [100, true],
  [199, true],
  [200, false],
])('at time %i the booking is active: %s', (now, expected) => {
  expect(isBookingActive(now, 100, 200)).toBe(expected);
});
```

Changing `< endsAt` to `<= endsAt`, or excluding the start, should fail. This protects a meaningful rule; do not remove it merely because it is a unit test. It does not establish timestamp parsing or timezone handling, which may need other tests.

## 6. Reliability: observe completion instead of guessing time

Promise: after a valid import finishes, all three accepted records are visible.

Weak:

```ts
await startImport(file);
await sleep(2000);
expect(await countRecords()).toBe(3);
```

This can fail on a slow worker and also counts unrelated records if state is shared. Raising the sleep or adding unconditional retries hides the synchronization problem.

Stronger:

```ts
const job = await startImport(file, { workspaceId: isolatedWorkspace.id });
await waitForJob(job.id, { status: 'completed', timeoutMs: 10000 });
const records = await readImportedRecords(job.id);
expect(records.map(record => record.externalId).sort())
  .toEqual(['row-1', 'row-2', 'row-3']);
```

The helper polls a read-only status with a deadline and useful diagnostics. It never resubmits the import. Missing a record fails deterministically after completion; a stuck import fails with the job state. The timeout follows the test environment's expected processing time and is not a universal value.
