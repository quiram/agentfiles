# Tailwind CSS Component Design Rules

Rules for using Tailwind CSS in a React/Next.js codebase so that styles remain maintainable, consistent with the design system, and free of duplication.

## Context

*Applies to:* All `.tsx` and `.ts` files in a Next.js / React project using Tailwind CSS
*Level:* Tactical/Operational — applies on every component or element written or reviewed
*Audience:* Developers and AI agents writing or reviewing Tailwind-styled React components

## Core Principles

1. *Components are the abstraction layer:* In a React + Tailwind codebase, the component is the reuse unit — not a CSS class. When the same visual pattern repeats, extract a component; do not create a semantic CSS class.
2. *Design tokens over raw values:* Colours and other design values defined in the project's Tailwind theme must be referenced by their token name, never by their raw value (hex, px, etc.).
3. *The theme is the source of truth:* If a value you need does not exist in the design token system, add it there — do not inline it.

## Rules

### Must Have (Critical)

- *RULE-001 — Extract repeated `className` strings into a component:* If the same `className` value (or a structurally identical combination of utilities) appears on more than one element, extract a component rather than copying the string. A `className` that would need to be updated in two places is duplication by the "one place to change" test.

- *RULE-002 — Use design token names, not raw values:* Whenever a colour or other design primitive is defined in the project's Tailwind theme, always reference it by its token class (e.g. `text-primary`, `bg-charcoal`) — never by its raw value (`text-[#4EB595]`, `bg-[#333333]`). The same applies to any other extended theme values (font families, breakpoints, etc.).

- *RULE-003 — Missing token → add to the theme, not inline:* If a design value you need is not yet in the Tailwind theme, add it to the project's theme configuration. Do not use an arbitrary value (e.g. `text-[#hex]`) as a substitute for a missing token.

### Should Have (Important)

- *RULE-101 — Prefer `children` over a `value` prop for variable content in extracted components:* When a repeated pattern wraps varying content, accept `children: React.ReactNode` rather than a typed `value` prop. This keeps the component flexible without unnecessary specialisation.

- *RULE-102 — Keep extracted components local until reuse is confirmed:* If a repeated pattern only appears within a single file, define the component locally at the top of that file. Promote it to `components/` only when a second file needs it.

### Could Have (Preferred)

- *RULE-201 — Avoid `@apply` for creating semantic classes:* Tailwind's `@apply` directive is intended for narrow use cases (e.g. base HTML element resets). Using it to create semantic CSS classes reintroduces the problems that utility-first CSS solves — hidden duplication, naming overhead, and loss of co-location. Prefer component extraction instead.

## Patterns & Anti-Patterns

### ✅ Do This

```tsx
// Repeated pattern → local component
function ScheduleRow({ label, children }: { label: string; children: React.ReactNode }): JSX.Element {
  return (
    <li>
      <span className="font-medium text-charcoal">{label}:</span> {children}
    </li>
  )
}

// Token name, not raw hex value
<span className="text-primary">Naturally</span>

// Missing token → add to theme config, then use by name
// tailwind.config.ts colors: { ..., ocean: '#2D5F8D' }
<span className="text-ocean">...</span>
```

### ❌ Don't Do This

```tsx
// Same className on multiple elements — extract a component
<li><span className="font-medium text-charcoal">Day:</span> Monday</li>
<li><span className="font-medium text-charcoal">Start:</span> 5 Jan</li>
<li><span className="font-medium text-charcoal">End:</span> 2 Mar</li>

// Raw hex instead of token name
<span className="text-[#4EB595]">Naturally</span>

// Arbitrary value for a colour that should be in the theme
<span className="text-[#2D5F8D]">...</span>
```

## Decision Framework

*When rules conflict:*
1. Extraction over duplication — a slightly more complex component is always better than copy-pasted `className` strings.
2. If a colour is genuinely a one-off (e.g. a third-party brand colour used once in an integration badge), an arbitrary value is acceptable — document why it is not a theme token.

*When facing edge cases:*
- A `className` that varies per-instance via conditional logic is not duplication — the conditional belongs inside the component, not the theme.
- A utility string used only once in a file is not a candidate for extraction, regardless of its length.

## Related Rules

- agents/agentfiles/platform/nextjs.rules.md — React component conventions for this codebase
- agents/agentfiles/design/avoiding-duplication.md — the "one place to change" test that drives RULE-001
- agents/agentfiles/design/parameterisation.md — `children` over `value` prop (RULE-101)

---

## TL;DR

*Key Principles:*
- Components, not CSS classes, are the abstraction unit in React + Tailwind
- Design token names always; raw values never
- Missing token → add to the theme config, not inline

*Critical Rules:*
- Repeated `className` string → extract a component (RULE-001)
- Use token names (`text-primary`) not raw values (`text-[#hex]`) (RULE-002)
- Missing value → add to theme config (RULE-003)

*Quick Decision Guide:*
Before copying a `className` string to a second element: extract a component. Before writing `text-[#...]`: check the theme — if the token exists, use it; if it doesn't, add it to the theme config.
