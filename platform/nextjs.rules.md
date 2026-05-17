# Next.js Platform Rules

Project-specific rules for the Next.js 15+ App Router with React 19 and TypeScript strict mode. Extends `agents/ee-llm-toolkit/rules/platform/typescript.rules.md` — these rules cover patterns that the generic TypeScript rules do not address.

## Context

*Applies to:* All `.tsx` and `.ts` files in this Next.js project — pages, layouts, components, and shared modules
*Level:* Tactical/Operational — applies on every component or exported function
*Audience:* All developers and AI agents working on this codebase

## Core Principles

1. *Explicit return types on exported components:* Pages, layouts, and any exported React component must declare a `JSX.Element` return type — this is the React 19 equivalent of TS-003 for components
2. *Import `JSX` explicitly:* React 19 dropped the global `JSX` namespace; any file using `JSX.Element` must import it as a type from `react`, or `tsc --noEmit` will fail even though ESLint passes

## Rules

### Must Have (Critical)

- *RULE-001:* Exported React components — pages, layouts, and any `export default function` or named exported component — must declare a `JSX.Element` return type. This is enforced by `@typescript-eslint/explicit-module-boundary-types` in `eslint.config.mjs` and surfaces in `npm run quality`.

- *RULE-002:* Any file that uses `JSX.Element` (or any member of the `JSX` namespace) must include `import type { JSX } from 'react'` at the top. In React 19 the global `JSX` namespace was removed; without the import, the file compiles through ESLint but `tsc --noEmit` fails. Both checks are part of `npm run quality` — fixing one without the other leaves the gate red.

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
// Missing return type — fails ESLint
export default function NotFound() {
  return <section>...</section>
}
```

```tsx
// Return type present but no JSX import — fails tsc --noEmit
import Link from 'next/link'

export default function NotFound(): JSX.Element {
  return <section><Link href="/">Home</Link></section>
}
```

## Quality Gates

- *Automated checks:* `@typescript-eslint/explicit-module-boundary-types: 'error'` in `eslint.config.mjs` flags missing return types; `tsc --noEmit` catches the missing `JSX` import. Both run under `npm run quality`.
- *Code review focus:* When reviewing new components, verify both the return type and the `import type { JSX } from 'react'` line are present.

## Related Rules

- agents/ee-llm-toolkit/rules/platform/typescript.rules.md — TS-003 (explicit return types for public functions); these Next.js rules apply the same principle to React components and add the React 19 import requirement
- agents/agentfiles/node/npm-quality.md — `npm run quality` must catch both violations

---

## TL;DR

*Critical Rules:*
- Exported React components declare `: JSX.Element` (RULE-001)
- Any file using `JSX.Element` must `import type { JSX } from 'react'` — React 19 removed the global namespace (RULE-002)

*Quick Decision Guide:*
Writing or editing a `.tsx` file with an exported component? Two things: explicit `JSX.Element` return type, and `import type { JSX } from 'react'` at the top. Both, or neither passes `npm run quality`.
