# Testing Anti-Patterns — 知行合一 Applied to Tests

Tests are how you prove knowledge (知) through action (行). But bad tests create false confidence — they pass without proving anything. This reference helps you recognize and avoid common testing traps.

## Anti-Pattern 1: Testing Mock Behavior Instead of Real Behavior

**Bad — tests the mock, not the system:**

```typescript
test('sends email on signup', async () => {
  const mockMailer = { send: jest.fn() };
  const service = new SignupService(mockMailer);

  await service.signup('user@test.com');

  // This only proves you called the mock, not that email actually works
  expect(mockMailer.send).toHaveBeenCalledWith('user@test.com');
});
```

**Good — tests the observable outcome:**

```typescript
test('sends email on signup', async () => {
  const testMailer = new InMemoryMailer(); // real implementation, in-memory
  const service = new SignupService(testMailer);

  await service.signup('user@test.com');

  expect(testMailer.sentEmails).toContainEqual(
    expect.objectContaining({ to: 'user@test.com' })
  );
});
```

**The principle:** If your test would pass even when the real system is broken, your test is testing the mock, not the system.

## Anti-Pattern 2: Test-Only Methods in Production Code

**Bad — production class has methods only tests use:**

```typescript
class UserCache {
  private cache = new Map();

  get(id: string) { return this.cache.get(id); }
  set(id: string, user: User) { this.cache.set(id, user); }

  // ✗ This exists only for tests
  _getInternalMap() { return this.cache; }
  _clearForTesting() { this.cache.clear(); }
}
```

**Good — test through the public interface:**

```typescript
test('caches user after first fetch', async () => {
  const cache = new UserCache();
  const user = { id: '1', name: 'Test' };

  cache.set('1', user);
  const result = cache.get('1');

  expect(result).toEqual(user);
});
```

**The principle:** If you need test-only methods, your public API is either insufficient or your test is testing internals.

## Anti-Pattern 3: Mocking Without Understanding Dependencies

**Bad — mocking everything to avoid understanding:**

```typescript
test('processes order', async () => {
  // 5 mocks = 5 things you don't understand
  const mockDB = jest.fn();
  const mockQueue = jest.fn();
  const mockPayment = jest.fn();
  const mockInventory = jest.fn();
  const mockNotifier = jest.fn();

  const service = new OrderService(mockDB, mockQueue, mockPayment, mockInventory, mockNotifier);
  await service.process(order);

  // You're testing that your code calls mocks in order, not that orders process correctly
});
```

**Good — use real implementations where feasible, mock only true external services:**

```typescript
test('processes order', async () => {
  const db = new TestDatabase();          // real DB, test instance
  const queue = new InMemoryQueue();      // real queue logic, in-memory
  const payment = new FakePaymentGateway(); // fake external service
  const inventory = new InventoryService(db); // real service, test DB
  const notifier = new InMemoryNotifier();

  const service = new OrderService(db, queue, payment, inventory, notifier);
  await service.process(order);

  expect(await db.getOrder(order.id)).toMatchObject({ status: 'processed' });
  expect(queue.messages).toContainEqual(expect.objectContaining({ type: 'order.processed' }));
});
```

**The principle:** Every mock is a lie about how the system works. Minimize lies.

## Anti-Pattern 4: Testing Implementation Instead of Behavior

**Bad — test breaks when refactoring internals:**

```typescript
test('validates email', () => {
  const validator = new EmailValidator();

  // Testing internal method call order
  const spy = jest.spyOn(validator, '_checkFormat');
  validator.validate('test@example.com');
  expect(spy).toHaveBeenCalledTimes(1);
});
```

**Good — test observable behavior:**

```typescript
test('rejects email without @', () => {
  const validator = new EmailValidator();
  expect(validator.validate('invalid')).toBe(false);
});

test('accepts standard email format', () => {
  const validator = new EmailValidator();
  expect(validator.validate('user@example.com')).toBe(true);
});
```

**The principle:** Tests should describe WHAT the system does, not HOW it does it internally. If you refactor the internals without changing behavior, tests should still pass.

## Anti-Pattern 5: Incomplete Mocks Hiding Real Bugs

**Bad — mock returns happy path, hides null handling bug:**

```typescript
test('displays user profile', async () => {
  const mockRepo = { findById: jest.fn().mockResolvedValue({ name: 'Test', email: 'a@b.com' }) };
  const component = render(<Profile repo={mockRepo} userId="1" />);

  // Always passes because mock never returns null
  expect(component.getByText('Test')).toBeDefined();

  // In production: findById returns null for deleted users → crash
});
```

**Good — test the unhappy path too:**

```typescript
test('shows fallback when user not found', async () => {
  const mockRepo = { findById: jest.fn().mockResolvedValue(null) };
  const component = render(<Profile repo={mockRepo} userId="deleted-user" />);

  expect(component.getByText('User not found')).toBeDefined();
});
```

**The principle:** If your mock only returns the happy path, you're only testing the happy path. Production has unhappy paths too.

## Quick Reference: When to Mock vs When Not To

| Dependency | Mock? | Why |
|-----------|-------|-----|
| External API (Stripe, AWS) | Yes | Unreliable, costly, slow |
| Database | Prefer test instance | Mocking SQL hides real query bugs |
| File system | Depends | In-memory FS for unit tests, real FS for integration |
| Internal service | No | Use the real implementation |
| Time/Date | Yes | Deterministic tests need controlled time |
| Random | Yes | Deterministic tests need controlled randomness |
