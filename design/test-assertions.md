# Test Assertions

Rules governing how many assertions a test may contain. Overrides `agents/ee-llm-toolkit/rules/testing-principles.md` RULE-001.

## Context

*Applies to:* All test files — unit, component, and integration
*Level:* Tactical/Operational — applies whenever a test is written or reviewed
*Audience:* All developers and AI agents

## Override Notice

This rule **overrides** RULE-001 of `agents/ee-llm-toolkit/rules/testing-principles.md`, which states:

> *RULE-001:* Each test must have a single, clear assertion about one specific behaviour

That rule is too narrow. A test covers a single **intent** — a single atomic piece of behaviour. Demonstrating that intent sometimes requires more than one assertion (e.g. checking that a returned object has the correct shape across several fields, or that a side effect produced both a database write and an emitted event). Forcing one assertion per test in those cases either splits a coherent intent across multiple test cases or hides assertions inside helpers that obscure what is actually verified.

This rule replaces RULE-001 wherever it conflicts.

## Core Principles

1. *Test the intent, not the assertion count:* What matters is that each test exercises one atomic, well-defined piece of behaviour — not how many `expect(...)` calls it contains.
2. *Cohesion over count:* Multiple assertions are fine when they all serve to demonstrate the same intent. They are not fine when they bundle unrelated behaviours into one test.
3. *One reason to fail (at the intent level):* A test should still fail for one logical reason — the intent it covers is broken. Multiple assertions that all fail together when the intent breaks satisfy this; multiple assertions that could fail independently for unrelated reasons do not.

## Rules

### Must Have (Critical)

- *RULE-001 — Tests may contain as many assertions as the intent requires:* A test is allowed multiple `expect(...)` calls when they collectively demonstrate a single atomic behaviour. There is no maximum assertion count.

- *RULE-002 — Each test still covers exactly one intent:* The relaxation on assertion count is **not** a licence to bundle unrelated behaviours. A test that asserts both "the invoice was created" and "the email was sent" covers two intents and must be split into two tests, even though both are properties of the same overall flow.

- *RULE-003 — The test name must describe the intent, not the assertions:* If you cannot give the test a single name that describes what it covers without using "and", the test is covering more than one intent and must be split.

### Should Have (Important)

- *RULE-101 — Group related assertions visually:* When multiple assertions support the same intent, place them together in the assert block with no intervening logic. If you find yourself interleaving assertions with arrange/act steps, that is a signal the test is doing too much.

- *RULE-102 — Prefer object-shape assertions when checking multiple fields of one result:* Asserting `expect(result).toEqual({ a: 1, b: 2, c: 3 })` is one assertion that covers three fields of a single intent. Prefer this to three separate `expect(result.a).toBe(1)` calls when all three fields belong to the same intent.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
// Single intent: "createInvoice returns a finalised invoice with the right shape"
// Multiple assertions all serve that one intent
it('createInvoice_whenContactExists_returnsFinalisedInvoiceForThatContact', async () => {
  const contact = buildContact({ url: 'contact/1' })
  const invoiceApi = stubInvoiceApi()

  const result = await createInvoice(contact.url, buildPayment())

  expect(result.contactUrl).toBe('contact/1')
  expect(result.status).toBe('Sent')
  expect(invoiceApi.markInvoiceAsSent).toHaveBeenCalledWith(result.url)
})
```

### ❌ Don't Do This

```typescript
// Two intents bundled — invoice creation AND email dispatch
it('processes payment correctly', async () => {
  await processPayment(payment)

  expect(invoiceApi.createInvoice).toHaveBeenCalled()  // intent 1: invoicing
  expect(emailApi.send).toHaveBeenCalled()              // intent 2: notification
})
// Split into two tests, one per intent.
```

## Decision Framework

*When in doubt:*
- Ask: "If this test fails, will it always be for the same reason?" If yes, the assertions belong together. If no, split the test.
- Ask: "Can I name this test without using 'and'?" If yes, the intent is single. If no, split.

## Related Rules

- agents/ee-llm-toolkit/rules/testing-principles.md — Base testing rules; this file overrides RULE-001 only
- agents/agentfiles/design/test-code-quality.md — Test code is subject to all design rules
- agents/agentfiles/design/naming-conventions.md — Test names describe intent

---

## TL;DR

*Overrides:* `ee-llm-toolkit/rules/testing-principles.md` RULE-001 (one assertion per test)

*Critical Rules:*
- A test may have as many assertions as its intent requires (RULE-001)
- Each test still covers exactly one atomic intent — no bundling unrelated behaviours (RULE-002)
- The test name must describe that intent without using "and" (RULE-003)

*Quick Decision Guide:*
Multiple assertions are fine when they all break together if the intent breaks. They are not fine when they could fail for independent reasons.
