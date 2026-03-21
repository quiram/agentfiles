# NPM Quality Script Convention

Rules for exposing linting and type checking as a single, discoverable npm command.

## Context

*Applies to:* All Node.js projects that have linting and/or type checking available
*Level:* Tactical/Operational — applies when setting up or modifying project scripts
*Audience:* All developers working on the project

## Core Principles

1. *Single entry point for quality checks:* Developers and CI should not need to know which individual tools (ESLint, tsc, etc.) are configured — one command covers all of them
2. *Complete coverage:* Quality checks must cover both production code and test code; silently skipping test files provides false confidence

## Rules

### Must Have (Critical)

- *RULE-001:* If linting or type checking is available, the project must expose an `npm run quality` script that runs all of them
- *RULE-002:* `npm run quality` must cover **both production and test code** — if separate tsconfig files exist for each, both type checks must be included
- *RULE-003:* `npm run quality` must not silently skip any configured quality tool

### Should Have (Important)

- *RULE-101:* Individual tool scripts (`lint`, `type-check`, `type-check:test`) may exist as separate commands but are called by `quality`, not instead of it
- *RULE-102:* CI runs `npm run quality` as a required step before running tests

## Patterns & Anti-Patterns

### ✅ Do This

```json
{
  "lint": "next lint",
  "type-check": "tsc --noEmit",
  "type-check:test": "tsc --noEmit --project tsconfig.test.json",
  "quality": "npm run lint && npm run type-check && npm run type-check:test"
}
```

### ❌ Don't Do This

```json
{
  "lint": "next lint",
  "type-check": "tsc --noEmit",
  // No quality script — developer has to know to run both manually
}
```

```json
{
  "quality": "npm run lint && npm run type-check"
  // Missing type-check:test — test code is never type-checked
}
```

## Quality Gates

- *Automated checks:* CI must include `npm run quality` as a step
- *Code review focus:* If a new tsconfig is introduced (e.g. for a new environment), verify `quality` is updated to include it

## Related Rules

- rules/npm-scripts.md — Conventions for npm script naming and the test command hierarchy

---

## TL;DR

*Critical Rules:*
- Always have `npm run quality` if linting or type checking exists
- It must cover both production and test code
- CI must run it

*Quick Decision Guide:*
If you add a new type-check scope or linter, add it to the `quality` script immediately.
