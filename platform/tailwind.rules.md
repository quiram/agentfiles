# Tailwind CSS Component Design Rules

Rules for using Tailwind CSS in a React/Next.js codebase so that styles remain maintainable, consistent with the design system, and free of duplication.

## Context

*Applies to:* All `.tsx` and `.ts` files in a Next.js / React project using Tailwind CSS
*Level:* Tactical/Operational — applies on every component or element written or reviewed
*Audience:* Developers and AI agents writing or reviewing Tailwind-styled React components

## Core Principles

1. *Components are the abstraction layer:* In a React + Tailwind codebase, the component is the reuse unit — not a CSS class. When the same visual pattern repeats, extract a component; do not create a semantic CSS class.
2. *Design tokens over raw values:* Colours and other design values defined in the project's Tailwind theme must be referenced by their token name, never by their raw value (hex, px, etc.).
3. *The palette is a design decision, not a coding decision:* Which colours the project uses is not something an agent decides unilaterally, in either direction — neither by reaching for a colour outside the palette, nor by adding a new one to the theme, nor by silently acting on a colour it finds already outside the palette. Any of those requires the user's judgement.

## Rules

### Must Have (Critical)

- *RULE-001 — Extract repeated `className` strings into a component:* If the same `className` value (or a structurally identical combination of utilities) appears on more than one element, extract a component rather than copying the string. A `className` that would need to be updated in two places is duplication by the "one place to change" test.

- *RULE-002 — Use design token names, not raw values:* Whenever a colour or other design primitive is defined in the project's Tailwind theme, always reference it by its token class (e.g. `text-primary`, `bg-charcoal`) — never by its raw value (`text-[#4EB595]`, `bg-[#333333]`). The same applies to any other extended theme values (font families, breakpoints, etc.).

- *RULE-003 — Need a colour outside the palette? Ask before using it, ask again before keeping it:* If the work genuinely calls for a colour the theme doesn't have, do not add it and do not inline it as an arbitrary value — stop and ask the user to confirm the colour itself first. If they confirm, that is permission to use it *this once* — it is not automatically permission to add it to the theme. Ask a second, separate question: should this become a permanent token, or is it a one-off? Only add it to the theme configuration if the user says yes to that second question.

- *RULE-004 — Found an existing colour outside the palette? Surface it, don't act on it:* If you encounter a raw value (`text-[#hex]`) or a theme token whose value doesn't trace back to the project's documented palette, do not decide what to do with it yourself — not tokenising it, not leaving it, not removing it. Report it to the user: where it is, what value it is, and what you found when you checked the project's design documentation for that exact value (present in one doc and absent or contradicted in another is common — say so explicitly if that's what you find). Let the user decide whether it's a legitimate exception, a mistake to fix, or a gap in the documented palette.

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

// Needed a colour the theme doesn't have — asked the user, they confirmed it
// AND confirmed it should be permanent, so it's added and used by name
// tailwind.config.ts colors: { ..., brandBlue: '#1E5A8A' }
<span className="text-brandBlue">...</span>

// Found `text-[#1E5A8A]` already in the codebase, tracing to no documented
// token — reported it to the user (with what the design docs said, or didn't
// say, about that value) instead of tokenising or removing it unasked
```

### ❌ Don't Do This

```tsx
// Same className on multiple elements — extract a component
<li><span className="font-medium text-charcoal">Day:</span> Monday</li>
<li><span className="font-medium text-charcoal">Start:</span> 5 Jan</li>
<li><span className="font-medium text-charcoal">End:</span> 2 Mar</li>

// Raw hex instead of token name
<span className="text-[#4EB595]">Naturally</span>

// Needed a colour the theme doesn't have, and added it to the theme
// unasked instead of confirming with the user first
// tailwind.config.ts colors: { ..., brandBlue: '#1E5A8A' }  ← nobody approved this

// Found a stray raw value already in the codebase and silently decided its
// fate — tokenising it, deleting it, or leaving it — without surfacing it
```

## Decision Framework

*When rules conflict:*
1. Extraction over duplication — a slightly more complex component is always better than copy-pasted `className` strings.
2. A colour genuinely used once (e.g. a third-party brand colour in an integration badge) may end up as an arbitrary value rather than a theme token — but that's still the outcome of asking under RULE-003, not a reason to skip asking. "It's just a one-off" is the answer to the second question (should this be a permanent token?), not a reason to bypass the first (can I use this colour at all?).

*When facing edge cases:*
- A `className` that varies per-instance via conditional logic is not duplication — the conditional belongs inside the component, not the theme.
- A utility string used only once in a file is not a candidate for extraction, regardless of its length.
- A colour the user has already confirmed earlier in the same conversation does not need re-confirming for the same piece of work — RULE-003 is about getting the decision made once, not about repeating the question.

## Related Rules

- agents/agentfiles/platform/nextjs.rules.md — React component conventions for this codebase
- agents/agentfiles/design/avoiding-duplication.md — the "one place to change" test that drives RULE-001
- agents/agentfiles/design/parameterisation.md — `children` over `value` prop (RULE-101)

---

## TL;DR

*Key Principles:*
- Components, not CSS classes, are the abstraction unit in React + Tailwind
- Design token names always; raw values never
- The palette is a design decision — never expand it, or act on a colour outside it, without asking the user

*Critical Rules:*
- Repeated `className` string → extract a component (RULE-001)
- Use token names (`text-primary`) not raw values (`text-[#hex]`) (RULE-002)
- Need a colour the theme doesn't have → ask to use it, then ask separately whether to keep it (RULE-003)
- Find a colour already outside the palette → report it, don't decide its fate yourself (RULE-004)

*Quick Decision Guide:*
Before copying a `className` string to a second element: extract a component. Before writing `text-[#...]` for a colour the theme doesn't have: ask the user, don't just add it or inline it. Before touching a stray colour you find already in the codebase: report what you found — including any contradiction across the project's design docs — and let the user decide.
