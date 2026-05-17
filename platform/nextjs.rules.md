# Next.js Platform Rules

Rules for Next.js 13+ App Router projects using React 19 and TypeScript. Extends `agents/ee-llm-toolkit/rules/platform/typescript.rules.md` (TS-003) — these rules cover Next.js/React 19 patterns the generic TypeScript rules do not address.

## Context

*Applies to:* All `.tsx` and `.ts` files in a Next.js App Router project on React 19
*Level:* Tactical/Operational — applies on every component or exported function
*Audience:* Developers and AI agents writing or reviewing Next.js components

## Core Principles

1. *Explicit return types on exported components:* Pages, layouts, and exported React components must declare a `JSX.Element` return type — the React 19 equivalent of TS-003 (explicit return types on public functions) applied to components
2. *Import `JSX` explicitly:* React 19 removed the global `JSX` namespace; any file using `JSX.Element` must import it as a type from `react`, or `tsc --noEmit` will fail even though ESLint passes

## Rules

### Must Have (Critical)

- *RULE-001:* Exported React components — pages, layouts, and any `export default function` or named exported component — must declare a `JSX.Element` return type. This applies TS-003 to components, where the return type would otherwise be inferred and silent.

- *RULE-002:* Any file that uses `JSX.Element` (or any member of the `JSX` namespace) must include `import type { JSX } from 'react'` at the top. In React 19 the global `JSX` namespace was removed; without the import, the file compiles through ESLint but `tsc --noEmit` fails. Both checks must be part of the project's quality gate — passing one without the other leaves the gate red.

## Patterns & Anti-Patterns

### ✅ Do This

```tsx
import type { JSX } from 'react'
import Link from 'next/link'

export default function NotFound(): JSX.Element {
  return (
    <section>
      <h1>Page not found</h1>
      <Link href="/">Home</Link>
    </section>
  )
}
```

```tsx
'use client'

import type { JSX } from 'react'

export default function HomePage(): JSX.Element {
  return <div>...</div>
}
```

### ❌ Don't Do This

```tsx
// Missing return type — fails @typescript-eslint/explicit-module-boundary-types
export default function NotFound() {
  return <section>...</section>
}
```

```tsx
// Return type present but no JSX import — fails tsc --noEmit on React 19
import Link from 'next/link'

export default function NotFound(): JSX.Element {
  return <section><Link href="/">Home</Link></section>
}
```

## Quality Gates

- *Automated checks:* `@typescript-eslint/explicit-module-boundary-types: 'error'` flags missing return types on exported functions; `tsc --noEmit` catches the missing `JSX` import. Both must run together as part of the project's quality gate.
- *Code review focus:* When reviewing new components, verify both the explicit return type and the `import type { JSX } from 'react'` line are present.

## Related Rules

- agents/ee-llm-toolkit/rules/platform/typescript.rules.md — TS-003 (explicit return types for public functions); these Next.js rules apply the same principle to React components and add the React 19 import requirement

---

## TL;DR

*Critical Rules:*
- Exported React components declare `: JSX.Element` (RULE-001)
- Any file using `JSX.Element` must `import type { JSX } from 'react'` — React 19 removed the global namespace (RULE-002)

*Quick Decision Guide:*
Writing or editing a `.tsx` file with an exported component? Two things: explicit `JSX.Element` return type, and `import type { JSX } from 'react'` at the top. Skipping either fails one of the two checks (ESLint or `tsc`).
