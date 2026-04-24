# Avoiding Duplication

Rules for identifying and eliminating duplication of knowledge — not just repeated lines, but repeated intent, logic, and responsibility. Based on Hunt & Thomas's DRY principle (*The Pragmatic Programmer*), Fowler's refactoring catalogue, and Kent Beck's preparatory refactoring discipline.

## Context

*Applies to:* All code — business logic, handlers, services, retry paths, and queue consumers
*Level:* Tactical/Operational — applies before writing any new implementation and when reviewing existing code
*Audience:* All developers

## Core Principles

1. *DRY is about knowledge, not lines:* Every piece of knowledge must have a single, authoritative representation in a system. Two implementations can look completely different and still duplicate the same knowledge.
2. *Business logic belongs in services, not in the things that trigger them:* Handlers, webhooks, and queue consumers are dumb pipes — they delegate, they do not implement.
3. *Make the change easy, then make the easy change:* When existing code makes reuse difficult, refactor first. Never duplicate because reuse feels hard right now.

## Rules

### Must Have (Critical)

- *RULE-001 — Use the "one place to change" test:* Before declaring an implementation correct, ask: if the logic for this operation changed, would I have to update it in more than one place? If yes, that is duplication — regardless of whether the code looks similar. Fowler calls the symptom "Shotgun Surgery": one logical change forces edits scattered across multiple locations. Consolidate before moving on.

- *RULE-002 — Business logic belongs in a service layer, not in the thing that triggers it:* Handlers, webhooks, controllers, and queue consumers are structural entry points — they translate input and delegate. Business operations (create invoice, send confirmation, confirm seat) must live in a shared service layer that any entry point can call. The diagnostic test: could this operation ever be needed from a second entry point? If yes, it does not belong inside the first entry point.

- *RULE-003 — Search before implementing:* Before writing any business operation, ask: does this already exist at a reusable layer? If yes, call it. If it exists but in the wrong place (buried in a handler or private to one module), extract it first — then call it from both places. Writing a second implementation of the same knowledge is never the right answer.

- *RULE-004 — Preparatory refactoring before the feature, not after:* When you find yourself thinking "I can't reuse this without duplicating it", stop — that thought is a signal to refactor, not a licence to duplicate. Following Kent Beck: *"for each desired change, make the change easy (warning: this may be hard), then make the easy change."* The preparatory refactoring (structural, no behaviour change) and the feature (behaviour change) must be in separate commits. If the refactoring is non-trivial, it should be a separate PR — or committed in a way that makes it easy to split into one later.

### Should Have (Important)

- *RULE-101 — Reach for idempotency when it eliminates a second implementation:* An idempotent operation — one that produces the same result whether called once or many times — can be invoked from both an initial-call path and a retry path without needing two different implementations. Use idempotency as a tool when it removes the need to maintain parallel logic for the same operation. Do not make operations idempotent unconditionally; make them idempotent when it collapses two code paths into one.

- *RULE-102 — Distinguish semantic duplication from incidental similarity:* Two pieces of code that look identical but represent different business rules are not duplicates — merging them would be wrong. Two pieces of code that look different but represent the same business rule are duplicates — leaving them separate is wrong. Apply the "one place to change" test (RULE-001) to tell them apart, not visual similarity.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
// Business operation lives in a service — callable from webhook and retry alike
// lib/freeagent/invoice-service.ts
export async function getOrCreateFinalizedInvoice(
  contactUrl: string,
  payment: PaymentRecord,
): Promise<Invoice> {
  const existing = await getInvoice(payment.stripePaymentIntentId)
  if (existing) return existing
  const invoice = await createInvoice({ contactUrl, ...payment })
  await markInvoiceAsSent(invoice.url)
  await markInvoiceAsPaid(invoice.url, payment)
  return invoice
}

// Webhook handler delegates — does not implement
async function processWebhookAsync(session: Stripe.Session): Promise<void> {
  const payment = buildPaymentRecord(session)
  const contact = await getOrCreateContact(payment)
  const invoice = await getOrCreateFinalizedInvoice(contact.url, payment)  // ← same function
  await sendCustomerEmail(payment, invoice)
}

// Retry handler also delegates — same function, no duplication
async function retryFreeAgentInvoice(payment: PaymentRecord): Promise<void> {
  const contact = await getOrCreateContact(payment)
  await getOrCreateFinalizedInvoice(contact.url, payment)  // ← same function
}
```

```typescript
// Preparatory refactoring: extract first (no behaviour change), then add the new caller
// Commit 1 — refactor: extract createAndFinalizeInvoice from webhook handler to service
// Commit 2 — feat: call createAndFinalizeInvoice from DLQ retry
```

### ❌ Don't Do This

```typescript
// Invoice logic duplicated across webhook and retry — two representations of the same knowledge
// app/api/webhooks/stripe/route.ts
async function processFreeAgentInvoice(emailData: CourseEmailData): Promise<void> {
  const contact = await getOrCreateContact(...)
  const existing = await findInvoiceByPaymentIntent(...)
  if (!existing) {
    const invoice = await createInvoice(...)
    await markInvoiceAsSent(invoice.url)
    await markInvoiceAsPaid(...)
  }
}

// lib/failed-events.ts — same knowledge, different implementation
async function retryFreeAgentInvoice(courseData: CourseEmailData): Promise<void> {
  const contact = await findOrCreateContact(...)  // re-implemented
  const existing = await findInvoiceByPaymentIntent(...)
  if (existing) {
    if (existing.status === 'Draft') await markInvoiceAsSent(...)
    if (...) await markInvoiceAsPaid(...)
    return
  }
  const invoice = await createInvoice(...)  // re-implemented
  await markInvoiceAsSent(invoice.url)
  await markInvoiceAsPaid(...)
}
// If the invoicing logic changes, it must be updated in two places.
// That is the definition of duplication.
```

## Decision Framework

*When rules conflict:*
1. Never duplicate to avoid a refactor — the short-term pain of refactoring is always less than the long-term pain of maintaining parallel implementations
2. If the preparatory refactoring feels too large to do now, make it a separate PR and track it as a debt item — but do not skip it

*When facing edge cases:*
- If two operations look similar but represent distinct business rules, leave them separate (RULE-102). Apply the "one place to change" test: if the rules could diverge independently, they are not duplicates.
- If extracting a shared service would require touching many files at once, use preparatory refactoring incrementally: extract one layer at a time, keeping tests green after each step.

## Related Rules

- design/code-organisation.md — function structure, DRY applied to conditional logic
- design/naming-conventions.md — a service named for its operation reveals where logic lives
- rules/clean-code.md — function size and single responsibility
- rules/solid-principles.md — SRP and DIP: depend on the service abstraction, not the handler

---

## TL;DR

*Key Principles:*
- DRY is about knowledge, not lines — if a change requires edits in two places, that is duplication
- Business operations belong in a service layer; handlers delegate, they do not implement
- Make the change easy first (refactor), then make the easy change (feature) — in separate commits

*Critical Rules:*
- Apply the "one place to change" test before shipping any implementation (RULE-001)
- Business logic must not live inside handlers or queue consumers (RULE-002)
- Search for an existing implementation before writing a new one (RULE-003)
- When existing code makes reuse hard, refactor first in a separate commit (RULE-004)

*Quick Decision Guide:*
When about to implement a business operation: (1) Does it already exist? Call it. (2) Does it exist in the wrong place? Extract it first, then call it. (3) Would making it idempotent eliminate a second implementation? Make it idempotent. (4) If any of these feel hard — that is the signal to refactor, not to duplicate.
