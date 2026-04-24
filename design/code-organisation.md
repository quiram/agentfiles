# Code Organisation

Rules for structuring functions and grouping operations so that code reads as a coherent narrative, at a consistent level of abstraction, without duplication of logic. Based on Robert C. Martin's Stepdown Rule, the Newspaper Metaphor, and the DRY / Once and Only Once principle.

## Context

*Applies to:* All functions, modules, and files — particularly multi-step handlers and orchestrators
*Level:* Tactical/Operational — applies when writing or reviewing functions with more than one step
*Audience:* All developers

## Core Principles

1. *Code is read top-to-bottom:* Organise so a reader can stop at any point and understand what they have read so far.
2. *One level of abstraction per function:* A function that mixes high-level orchestration with low-level detail forces readers to context-switch mid-thought.
3. *Knowledge lives in one place:* Duplicated logic — including duplicated conditional checks — is as harmful as duplicated code.

## Rules

### Must Have (Critical)

- *RULE-001 — The Newspaper Metaphor:* Organise source files and functions from highest to lowest abstraction. Public or top-level functions appear first; private helpers appear immediately below the function that uses them; low-level utilities appear last. A reader scanning top-to-bottom moves from intent to detail, not back and forth.

- *RULE-002 — Single Level of Abstraction per function:* Every statement in a function must operate at the same level of abstraction. If a function calls `sendCustomerEmail(data)` (high-level) and also executes `Buffer.from(base64, 'base64')` (low-level) in the same body, extract the low-level step into a named helper. The test: can every line in the function be described in the same sentence as the function's name?

- *RULE-003 — A function's name is a contract about everything it does:* If `processPaymentEmails` also creates a FreeAgent invoice, the name is lying. Rename it to surface all responsibilities (`processPaymentNotificationsAndInvoice`) — the uncomfortable name reveals the SRP violation and prompts a split. Never hide a responsibility inside a function named for a different one.

- *RULE-004 — DRY applies to logic, not just code:* A conditional check represents knowledge: "does X exist?" Checking the same condition twice in sequence duplicates that knowledge. The second check must be inside the first's branch, or both checks must be collapsed into a named abstraction (e.g. `getOrCreate`). Duplicated conditional logic is harder to spot than duplicated code but equally harmful.

### Should Have (Important)

- *RULE-101 — Related operations belong at the same call-stack layer:* Operations that share a concern should be invoked from the same place. If two FreeAgent calls (`getOrCreateContact` and `createInvoice`) serve the same accounting concern, they belong in the same function or called from the same orchestrator — not one inside an email function and the other in a webhook handler.

- *RULE-102 — Apply patterns consistently:* If a pattern (e.g. `getOrCreate` for idempotent resource creation) is used for one resource type, apply it to all resources in the same file with the same requirement. Inconsistency signals the pattern was applied reactively. Before completing any function that creates a resource, ask: does an equivalent pattern already exist elsewhere in this file?

- *RULE-103 — Orchestrators read as ordered steps, not as implementation:* A top-level handler function should read like a numbered checklist of concerns. If following that list requires reading into a sub-function's body to understand what the step does, the abstraction boundary is in the wrong place and the detail is leaking upward.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
// Top-level orchestrator reads as a checklist — each step is one concern
async function processWebhookAsync(session: Stripe.Session): Promise<void> {
  const payment = buildPaymentContext(session)
  if (await isAlreadyProcessed(payment)) return
  const invoicePdf = await processAccountingRecord(payment)
  await sendCustomerEmail(payment, invoicePdf)
  await sendAdminEmail(payment)
}

// DRY: knowledge of "did the invoice exist?" lives in one place
async function getOrCreateInvoice(contactUrl: string, payment: PaymentContext): Promise<Invoice> {
  const existing = await getInvoice(payment.stripePaymentIntentId)
  if (existing) return existing
  const invoice = await createInvoice(contactUrl, payment)
  await markInvoiceAsSent(invoice.url)
  await markInvoiceAsPaid(invoice.url, payment)
  return invoice
}
```

### ❌ Don't Do This

```typescript
// Orchestrator hides an accounting concern inside an email function
async function processPaymentEmails(payment: PaymentContext): Promise<void> {
  const invoicePdf = await processFreeAgentInvoice(payment)  // ← wrong layer
  await sendCustomerEmail(payment, invoicePdf)
  await sendAdminEmail(payment)
}

// DRY violation: existingInvoice checked twice; second check is outside the first's branch
async function processFreeAgentInvoice(payment: PaymentContext): Promise<Buffer | null> {
  const existingInvoice = await getInvoice(payment.stripePaymentIntentId)
  let invoice = existingInvoice
  if (!existingInvoice) {
    invoice = await createInvoice(contactUrl, payment)
  }
  if (!existingInvoice) {  // ← knowledge duplicated; should be inside the block above
    await markInvoiceAsSent(invoice!.url)
    await markInvoiceAsPaid(invoice!.url, payment)
  }
  return fetchInvoicePdf(invoice!.url)
}
```

## Decision Framework

*When rules conflict:*
1. Abstraction consistency takes priority over function brevity — a slightly longer function at a consistent abstraction level is better than a short one that mixes levels
2. When two concerns appear in the same function and cannot be separated cleanly, treat it as a signal to redesign the data model, not to accept the mixed function

*When facing edge cases:*
- If lifting a call to a shared orchestrator would require duplicating setup logic, extract that setup into a shared helper rather than keeping the call in the wrong place
- If a `getOrCreate` abstraction is genuinely not reusable across two similar resources, still prefer two separate `getOrCreate` functions over inline duplication of the check

## Related Rules

- rules/clean-code.md — Function size and single responsibility
- rules/solid-principles.md — SRP: one reason to change
- design/naming-conventions.md — A function's name constrains what it should do

---

## TL;DR

*Key Principles:*
- Code reads top-to-bottom: highest abstraction first, implementation detail below
- Every statement in a function operates at the same level of abstraction
- Duplicated conditional logic is as harmful as duplicated code

*Critical Rules:*
- Organise files high-to-low: public functions first, helpers below (RULE-001)
- One level of abstraction per function — extract any line that breaks it (RULE-002)
- A function's name is a contract about everything it does — a lying name reveals an SRP violation (RULE-003)
- Never check the same condition twice in sequence — collapse or extract (RULE-004)

*Quick Decision Guide:*
Read each function top-to-bottom and ask: (1) Does every line belong to the same concern? (2) Is every line at the same abstraction level? (3) Is any conditional check repeated? If any answer is no, refactor before moving on.
