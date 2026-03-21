# Test Suite Organisation

Rules for structuring and locating test files within a project. Governs how tests are classified, where they live, and what naming conventions to follow.

## Context

*Applies to:* All projects with automated test suites using Jest or equivalent
*Level:* Tactical/Operational — guides daily test writing and CI configuration
*Audience:* All developers writing or reviewing tests

## Core Principles

1. *Classification by dependency boundary:* A test's location is determined by whether it crosses an external dependency boundary (network, database, filesystem, third-party API) — not by what it tests
2. *Predictable location:* Any developer should be able to find a test file without searching — the folder tells them what kind of test it is
3. *Independent runnability:* Unit and integration test suites must be runnable independently of each other; integration tests should not be required for a fast local feedback loop

## Rules

### Must Have (Critical)

- *RULE-001:* All tests live under `__tests__/` at the project root, split into `__tests__/unit/`, `__tests__/integration/`, and `__tests__/components/`
- *RULE-002:* A test is a **unit test** if it mocks all external dependencies (databases, APIs, network). It goes in `__tests__/unit/`
- *RULE-003:* A test is an **integration test** if it calls a real external service (Stripe, Resend, Neon/Prisma against a real DB). It goes in `__tests__/integration/`
- *RULE-004:* Component tests (React, jsdom) that mock all network calls go in `__tests__/components/`
- *RULE-005:* Test file names mirror the source file they cover: `lib/freeagent/contacts.ts` → `__tests__/unit/freeagent-contacts.test.ts`

### Should Have (Important)

- *RULE-101:* Integration tests must auto-skip when their required environment variable is absent (e.g. `DATABASE_URL`, `RESEND_API_KEY`) — never fail hard in environments where the service is unavailable
- *RULE-102:* ALL tests (unit, component, and integration) run on every push to any branch — `npm test` is the CI command. Running only unit and component tests is acceptable between local commits as a fast feedback loop, not as a substitute for the full suite
- *RULE-103:* There must be no duplicate test files covering the same unit — consolidate before moving

### Could Have (Preferred)

- *RULE-201:* Name integration test files with a suffix that signals the external dependency: `.db.test.ts` for database tests, `.integration.test.ts` for API tests

## Patterns & Anti-Patterns

### ✅ Do This

```
__tests__/
  unit/
    checkout-api.test.ts        # mocks Stripe
    reservations.test.ts        # mocks Prisma
    freeagent-contacts.test.ts  # mocks freeAgentFetch
  components/
    BuyNowButton.test.tsx       # jsdom + mocked fetch
  integration/
    checkout-integration.test.ts    # real Stripe test mode
    reservations.db.test.ts         # real Neon dev branch
    email-integration.test.ts       # real Resend API
```

### ❌ Don't Do This

```
__tests__/
  buy-now-button.test.tsx     # flat, no classification
  BuyNowButton.test.tsx       # duplicate of the above
  reservations.db.test.ts     # integration test mixed with unit tests
  checkout-api.test.ts        # unclear if unit or integration
```

## Decision Framework

*When rules conflict:*
1. If a test mocks some externals but hits one real service → it is an integration test
2. If unsure, ask: "Does this test require network access or a running service?" — yes → integration, no → unit
3. When consolidating duplicates, keep the file with better test names and coverage; absorb any unique cases from the other

*When facing edge cases:*
- Tests using `msw` to intercept HTTP at the network layer are still unit tests (no real service hit)
- Tests using an in-memory SQLite or PGlite database are unit tests (no real external DB)
- Tests that skip based on an env var but would hit a real service if the var were present are integration tests

## Exceptions & Waivers

*Valid reasons for exceptions:*
- Framework-colocated tests (e.g. `*.spec.ts` next to source files) — only if the project convention predates these rules
- Performance/load tests that don't fit either category — place in `__tests__/perf/`

*Process for exceptions:*
1. Document why the test cannot follow the folder rule in a comment at the top of the file
2. Create a follow-up task to normalise once the constraint is resolved

## Quality Gates

- *Automated checks:* Jest `testMatch` config must explicitly point at `__tests__/unit/**`, `__tests__/components/**`, and `__tests__/integration/**` — no loose glob that catches all three indiscriminately
- *Code review focus:* Verify new test files are in the correct subfolder before approving; flag any test that hits a real service but lives in `unit/` or `components/`
- *Testing requirements:* After any reorganisation, run all three suites independently to confirm nothing is broken or silently skipped

## Related Rules

- rules/testing-principles.md — Core testing philosophy (test pyramid, isolation, naming)
- rules/task-execution.md — Tests must pass before marking a task complete

---

## TL;DR

*Key Principles:*
- Folder = dependency boundary: mocks → `unit/`, real services → `integration/`, React/jsdom → `components/`
- All tests run on every push; unit+component only is acceptable between local commits
- Unit and integration suites must run independently

*Critical Rules:*
- `__tests__/unit/` for tests with all externals mocked
- `__tests__/integration/` for tests hitting real Stripe, Resend, or DB
- `__tests__/components/` for React component tests (jsdom + mocked network)
- Integration tests must auto-skip when env vars are absent
- No duplicate test files

*Quick Decision Guide:*
When in doubt: does this test require a network call or a running service? Yes → `integration/`. No → `unit/` or `components/`.
