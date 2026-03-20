# NPM Script Conventions

Rules for naming and behaviour of npm scripts, particularly around testing and builds. Consistent script names prevent confusion and ensure the master build runs the right things.

## Context

*Applies to:* All projects using npm (or compatible package managers) with a `package.json`
*Level:* Tactical/Operational — applies when adding or modifying npm scripts
*Audience:* All developers working on the project

## Core Principles

1. *`npm test` is the master gate:* The canonical test command must run everything needed to assert the project is correct — it is what CI and builds rely on
2. *Narrower commands for developer convenience:* Scoped scripts (unit-only, etc.) exist as fast-feedback tools during development, not as substitutes for the full suite
3. *Predictable naming:* Script names follow a consistent hierarchy so developers can discover them without reading documentation

## Rules

### Must Have (Critical)

- *RULE-001:* `npm test` (i.e. the `"test"` script) runs the **complete test suite** — unit, component, and integration tests. This is the command used in master builds and CI
- *RULE-002:* `npm run test:unit` runs only unit and component tests — the fast subset suitable for running between commits during local development
- *RULE-003:* Never configure `npm test` to run only a subset of tests; doing so would silently exclude integration tests from the master build

### Should Have (Important)

- *RULE-101:* Integration test scripts follow the naming convention `test:integration`, `test:integration:db`, `test:integration:api`, etc.
- *RULE-102:* All test scripts that require environment variables must auto-skip gracefully when those variables are absent — never fail hard
- *RULE-103:* CI runs `npm test` on every push to every branch; `npm run test:unit` is only a local convenience tool

## Patterns & Anti-Patterns

### ✅ Do This

```json
{
  "test": "npm run test:unit && npm run test:integration",
  "test:unit": "jest --config jest.config.js",
  "test:integration": "npm run test:integration:api && npm run test:integration:db",
  "test:integration:api": "jest --config jest.integration.config.js",
  "test:integration:db": "jest --config jest.db.config.js"
}
```

### ❌ Don't Do This

```json
{
  "test": "jest --config jest.unit.config.js",  // excludes integration
  "test:all": "jest"                             // wrong name for the complete suite
}
```

## Quality Gates

- *Automated checks:* CI pipeline always invokes `npm test` — never a scoped variant like `test:unit`
- *Code review focus:* If a PR modifies the `"test"` script, verify it still runs the complete suite

## Related Rules

- rules/test-organisation.md — How tests are structured and classified

---

## TL;DR

*Critical Rules:*
- `npm test` = everything (master build command)
- `npm run test:unit` = unit + component only (fast local loop)
- Never let `npm test` run less than the full suite

*Quick Decision Guide:*
Developing locally between commits → `npm run test:unit`. Pushing / CI / build → `npm test`.
