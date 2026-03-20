# Alias Imports

Rules for how modules are imported within a TypeScript project that configures path aliases.

## Context

*Applies to:* All TypeScript and TSX files in projects with `@/` path alias configured in `tsconfig.json`
*Level:* Tactical/Operational — applies on every import statement written
*Audience:* All developers writing TypeScript in this codebase

## Core Principles

1. *Consistency:* All internal imports use the same form; a reader never has to count `../` levels
2. *Refactoring safety:* Alias imports do not break when files are moved to a different depth in the directory tree
3. *Clarity:* `@/lib/stripe` immediately tells you where the module lives; `../../lib/stripe` does not

## Rules

### Must Have (Critical)

- *RULE-001:* Always import project modules using the `@/` alias: `@/lib/foo`, `@/components/Bar`, `@/app/api/baz/route`
- *RULE-002:* Never use relative imports (`../`, `../../`, etc.) for project modules — including in test files, scripts, and files outside `app/`
- *RULE-003:* `jest.mock()` calls must also use the `@/` alias, not relative paths

### Should Have (Important)

- *RULE-101:* Third-party packages and Node built-ins are always bare imports (`stripe`, `next/server`, `fs`) — never alias them
- *RULE-102:* When moving a file to a different directory, verify no relative imports were introduced — run a grep for `from '\.\./` as a sanity check

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
import { getStripe } from '@/lib/stripe'
import { CourseFullError } from '@/lib/reservations'
import { BuyNowButton } from '@/components/BuyNowButton'
import { POST } from '@/app/api/checkout/route'

jest.mock('@/lib/stripe', () => ({ ... }))
```

### ❌ Don't Do This

```typescript
import { getStripe } from '../../lib/stripe'
import { CourseFullError } from '../lib/reservations'
import { POST } from '../app/api/checkout/route'

jest.mock('../lib/stripe', () => ({ ... }))
```

## Quality Gates

- *Automated checks:* ESLint `no-restricted-imports` rule can enforce this if configured
- *Code review focus:* Flag any `from '\.\./` or `from '\.\/` that imports a project module rather than a relative sibling config file

## Related Rules

- rules/platform/typescript.rules.md — TypeScript conventions for this codebase

---

## TL;DR

*Critical Rules:*
- Use `@/lib/...`, `@/components/...`, `@/app/...` for all project imports
- No `../` or `../../` for project modules — anywhere, including tests and `jest.mock()`
- Third-party and Node built-ins stay as bare imports

*Quick Decision Guide:*
When in doubt: if it lives inside this repo, use `@/`. If it came from `npm install`, use the bare name.
