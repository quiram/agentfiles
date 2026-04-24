# Test Code Quality

Test code is code. All design rules — naming conventions, code organisation, avoiding duplication — apply to tests with the same force as to production code.

## Context

*Applies to:* All test files — unit, component, and integration
*Level:* Tactical/Operational — applies when writing or reviewing any test
*Audience:* All developers

## Core Principles

1. *Tests are maintained, therefore they must be maintainable:* Test code is read, modified, and debugged by humans. The same standards of naming, organisation, and clarity that apply to production code apply here.
2. *A bad test is worse than no test:* An unreadable or poorly structured test gives false confidence and creates maintenance burden without adding safety.
3. *DRY applies to tests:* Duplicated setup, duplicated assertion logic, and duplicated knowledge in tests are the same problem as in production code — apply the "one place to change" test equally.

## Rules

### Must Have (Critical)

- *RULE-001 — Test code is subject to all design rules:* Naming conventions (`design/naming-conventions.md`), code organisation (`design/code-organisation.md`), and the duplication rules (`design/avoiding-duplication.md`) apply to test files without exception. A test file is not a place to relax standards.

- *RULE-002 — Test names describe the scenario and expected outcome:* A test name is documentation. It must state the condition under test and what should happen — not the implementation detail being exercised. Use a consistent pattern across the suite: `given_X_when_Y_then_Z`, `methodName_scenario_expectedResult`, or `should_doX_when_Y` — pick one and use it everywhere.
  - Bad: `test1`, `it works`, `handlePayment`
  - Good: `createInvoice_whenContactExists_skipsContactCreation`, `should_confirmSeat_when_paymentSucceeds`

- *RULE-003 — Extract shared setup and assertion mechanics:* Duplicated arrange blocks, repeated object construction, and copy-pasted assertion patterns are duplication. Extract them into named helpers. The test body should express *what* is being tested; shared mechanics of *how* tests are assembled belong in helpers.

- *RULE-004 — Follow the red-green cycle:* Write the test first, confirm it fails (red), then write the minimum implementation to make it pass (green). Never write implementation code before a failing test exists for the behaviour it introduces. A test that was never red provides no assurance — it may be passing vacuously or testing the wrong thing.

### Should Have (Important)

- *RULE-101 — Each test has one reason to fail:* A test that asserts ten things will fail for ten different reasons. One clear assertion per test makes failures diagnostic. If a test legitimately needs to verify multiple related properties, group them under a single named concept and assert that concept.

- *RULE-102 — Avoid logic in tests:* Conditionals, loops, and computed expected values inside a test body are a smell — they mean the test is either testing multiple scenarios or computing its own expected values, both of which obscure what is actually verified. Extract separate tests or pre-compute expected values as named constants.

- *RULE-103 — FIRST principles apply:* Tests must be Fast (run in milliseconds), Independent (no shared mutable state between tests), Repeatable (same result on every run), Self-validating (pass/fail without manual inspection), and Timely (written alongside the code they test).

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
// Shared construction extracted — not duplicated across every test
function buildPayment(overrides: Partial<PaymentRecord> = {}): PaymentRecord {
  return { paymentIntentId: 'pi_test', coursePrice: 100, courseSlug: 'a1-3', ...overrides }
}

// Test body focuses on what makes this scenario distinct
it('createInvoice_whenInvoiceAlreadyExists_returnsExistingWithoutCreatingNew', async () => {
  const existing = buildInvoice({ paymentIntentId: 'pi_123' })
  const invoiceApi = stubInvoiceApi({ existing })

  const result = await getOrCreateInvoice('contact/1', buildPayment({ paymentIntentId: 'pi_123' }))

  expect(result).toEqual(existing)
  expect(invoiceApi.createInvoice).not.toHaveBeenCalled()
})
```

### ❌ Don't Do This

```typescript
// Duplicated setup — same 8-line object copy-pasted across multiple tests
it('sends email on success', async () => {
  const session = { id: 's_1', customer: 'cus_1', payment_intent: 'pi_1', amount_total: 10000, ... }
  ...
})
it('creates invoice on success', async () => {
  const session = { id: 's_1', customer: 'cus_1', payment_intent: 'pi_1', amount_total: 10000, ... }
  // identical to above — if the shape changes, update in two places
})

// Logic inside the test — computes its own expected value
it('calculates correct total', async () => {
  const items = getTestItems()
  const expected = items.reduce((sum, i) => sum + i.price, 0)  // if calculateTotal is wrong, this may be too
  expect(calculateTotal(items)).toBe(expected)
})

// Vague test name — fails to document the scenario
it('handles existing invoice', async () => { ... })
```

## Decision Framework

*When rules conflict:*
1. Readability of what is being tested wins — but the solution is good naming and focused helpers, never duplicated code
2. If making a test DRY requires extracting logic that obscures what the test asserts, the extraction needs a better name, not less extraction

*When facing edge cases:*
- Integration tests that share expensive setup (e.g. database seeding) may share `beforeAll` blocks for performance — acceptable exception to independence, not an exception to naming or DRY
- If applying DRY to tests reveals that the same scenario is being tested in multiple places, consolidate the tests just as you would consolidate production code

## Related Rules

- design/naming-conventions.md — applies fully to test files, helper names, and test suite names
- design/code-organisation.md — applies fully; tests read top-to-bottom like production code
- design/avoiding-duplication.md — the "one place to change" test applies to test code equally
- rules/testing-principles.md — core test pyramid, isolation, and AAA structure

---

## TL;DR

*Key Principles:*
- Test code is code — all design rules apply without exception
- DRY applies: duplicated setup, assertion logic, and knowledge in tests must be extracted
- Test names are documentation — they describe the condition and expected outcome

*Critical Rules:*
- All design rules apply to tests: naming, organisation, duplication (RULE-001)
- Test names state the scenario and expected outcome in a consistent pattern (RULE-002)
- Shared setup and assertion mechanics must be extracted into helpers, not duplicated (RULE-003)
- Write the test first, confirm it fails, then implement — never write code without a red test first (RULE-004)

*Quick Decision Guide:*
If the logic for test setup changed, would you update it in more than one place? If yes, extract it. Is the test name clear about what fails when the test fails? If not, rename it.
