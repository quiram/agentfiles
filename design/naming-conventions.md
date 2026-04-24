# Naming Conventions

Rules for naming types, functions, variables, and constants so that code is self-documenting, consistent, and free of misleading labels. Based on Robert C. Martin's *Clean Code* (chapters 2 and 17), the Google TypeScript Style Guide, and the mkosir TypeScript Style Guide.

## Context

*Applies to:* All code — types, interfaces, functions, variables, constants, booleans
*Level:* Tactical/Operational — applies on every identifier written or reviewed
*Audience:* All developers

## Core Principles

1. *Names are contracts:* A name commits to what a thing is, contains, or does. A name that no longer matches its subject is a lie.
2. *Reveal intent:* If a name requires a comment to explain it, the name is wrong.
3. *One word per concept:* Pick a verb or noun for each concept and use it everywhere — inconsistency forces readers to mentally map synonyms.

## Rules

### Must Have (Critical)

- *RULE-001 — Avoid noise words:* Words like `Data`, `Info`, `Manager`, `Processor`, `Handler`, and `Object` add no meaning. They signal that the author could not name the thing precisely. Before appending a noise word, ask what the name would mean without it — that shorter name is almost always better.
  - Bad: `CourseEmailData`, `UserInfo`, `PaymentManager`, `EventHandler`
  - Good: `CourseEmail`, `User`, `PaymentService`, `onPaymentReceived`

- *RULE-002 — Names must match content, not intended use:* A name reflects what a thing actually contains or does, not the context in which it was first created. If a type named `CourseEmail` grows to carry fields that no email template reads, the name is lying. Either remove those fields or rename the type to reflect its full scope.

- *RULE-003 — Use parts of speech correctly:*
  - Types and classes: nouns or noun phrases (`Invoice`, `CustomerAddress`, `PaymentContext`)
  - Functions and methods: verbs or verb phrases (`createInvoice`, `markAsSent`, `sendConfirmation`)
  - Booleans: positive predicates (`isConfirmed`, `hasInvoice`, `canRetry`) — avoid negative predicates; `if (!isNotSent)` is a double negative that obscures intent

- *RULE-004 — One word per concept:* Choose one verb per operation type and use it consistently across the whole codebase. Do not mix `fetch`, `retrieve`, `get`, and `find` for the same conceptual operation.
  - Bad: `fetchUser()`, `retrieveOrder()`, `getAccount()`, `findInvoice()`
  - Good: `getUser()`, `getOrder()`, `getAccount()`, `getInvoice()`

### Should Have (Important)

- *RULE-101 — Reveal intent; eliminate the need for comments:* A variable named `d` with a comment `// elapsed time in days` should be named `elapsedTimeInDays`. The name should answer why the variable exists, what it holds, and how it is used.

- *RULE-102 — Avoid disinformation:* Never name a collection `accountList` if it is a `Set` or `Map`. Never reuse a name across two different concepts in the same scope. Readers trust names; a misleading name costs more time than a verbose one.

- *RULE-103 — Prefer pronounceable, searchable names:* Avoid abbreviations and single-letter variables except in the smallest scopes (e.g. loop indices). `generationTimestamp` is searchable; `genymdhms` is not.

### TypeScript-Specific (Critical)

- *RULE-TS-001:* Do not prefix interfaces or types with `I` (`IOrder` → `Order`). Modern TypeScript does not require this convention and it adds noise.
- *RULE-TS-002:* Generic type parameters must be `T`-prefixed and descriptive (`TRequest`, `TPayload`). Single-letter generics (`T`, `K`, `U`) obscure what the parameter represents.
- *RULE-TS-003:* Treat acronyms as words: `loadHttpUrl`, not `loadHTTPURL`; `parseJson`, not `parseJSON`.
- *RULE-TS-004:* Prefer `type` over `interface` for all type definitions unless declaration merging is required.
- *RULE-TS-005:* `CONSTANT_CASE` applies only to module-level immutable values and enum members. Local `const` variables use `camelCase`.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
// Type name matches all of its content
type PaymentContext = {
  customerEmail: string
  customerName: string
  courseTitle: string
  coursePrice: number
  stripePaymentIntentId: string  // clearly part of a payment context
  customerAddress: CustomerAddress
}

// Functions named consistently with one verb per concept
async function getContact(email: string): Promise<Contact | null> { ... }
async function getInvoice(paymentIntentId: string): Promise<Invoice | null> { ... }

// Boolean as positive predicate
const isConfirmed = reservation.status === 'CONFIRMED'
if (isConfirmed) return  // reads naturally
```

### ❌ Don't Do This

```typescript
// Noise word "Data" — what does CourseEmailData mean that CourseEmail doesn't?
type CourseEmailData = {
  customerEmail: string
  courseTitle: string
  stripePaymentIntentId: string  // nothing to do with emails
  customerAddress: CustomerAddress  // nothing to do with emails
}

// Inconsistent verbs for the same operation
async function getContact(email: string) { ... }
async function findInvoiceByPaymentIntent(id: string) { ... }  // "find" vs "get"

// Negative boolean predicate
const isNotSent = invoice.status !== 'sent'
if (!isNotSent) { ... }  // double negative — what does this mean?
```

## Decision Framework

*When rules conflict:*
1. Accuracy over brevity — a longer, accurate name is always better than a shorter, misleading one
2. If a noise word is the only way to disambiguate two genuinely different concepts, reconsider whether the concepts themselves are well-separated

*When facing edge cases:*
- If renaming a type reveals that it contains unrelated fields, that is a signal to split the type, not to keep the bad name
- If one word per concept conflicts with an established library or framework term, prefer the established term within that bounded context

## Related Rules

- rules/clean-code.md — General readability and simplicity principles
- design/parameterisation.md — Naming of parameters specifically
- design/code-organisation.md — How naming interacts with function structure

---

## TL;DR

*Key Principles:*
- A name is a contract about what something is, contains, or does — misleading names cost more than verbose ones
- Noise words (`Data`, `Info`, `Manager`) signal imprecision; remove them
- One word per concept, used consistently across the whole codebase

*Critical Rules:*
- No noise words; ask what the name means without the suffix (RULE-001)
- Name must match content, not intended use (RULE-002)
- Types=nouns, functions=verbs, booleans=positive predicates (RULE-003)
- One verb per concept, used consistently (RULE-004)

*Quick Decision Guide:*
Before naming anything: what is this thing, precisely? If the answer requires more than one sentence, split the thing. If the name you choose requires a comment to explain it, choose a better name.
