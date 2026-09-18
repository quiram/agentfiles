# Illegal States

Rules for shaping data types so that invalid combinations of fields cannot be constructed, rather than relying on runtime checks or code review to catch them. Based on the "make illegal states unrepresentable" principle from typed functional programming.

## Context

*Applies to:* Any type, interface, or schema with two or more fields whose validity depends on each other
*Level:* Tactical/Operational — applies when defining a new data shape or reviewing one with optional fields
*Audience:* All developers working in a statically typed language

## Core Principles

1. *A type is a promise:* Every value the type system allows a caller to construct is a value the rest of the code must handle correctly. A type that allows combinations the domain forbids forces every consumer to re-derive and re-check the real constraint by hand.
2. *Validation at construction beats validation at use:* If a shape can only be used safely one way, encode that in the shape itself. Don't leave it to a runtime `if` that every call site must remember to write.
3. *A shape with no valid encoding for "neither" or "both" cannot silently fall through to a case nobody wrote.*

## Rules

### Must Have (Critical)

- *RULE-001 — Mutually exclusive fields must be a union, not independent optionals:* If two or more fields are only ever valid in specific combinations (exactly one of A or B; A implies C; etc.), do not model them as independent optional fields on one flat type. Use a discriminated union (a tagged variant in TypeScript; a sum type, sealed class hierarchy, or enum-with-payload in other typed languages) so the combinations the domain forbids have no representation at all.

- *RULE-002 — A discriminant field earns its keep by driving control flow:* When you add a union, branch on its discriminant (`kind`, `type`, `status`, ...) wherever the union is consumed. Do not re-derive the variant by checking which optional field happens to be set — that reintroduces the ambiguity the union was meant to remove, and duplicates the same "which case is this?" check at every call site.

- *RULE-003 — No catch-all branch for a state the type no longer allows:* Once a union rules out "neither" or "both", the code that consumes it must not keep a fallback branch for that state (e.g. rendering nothing, throwing a generic error). An exhaustive switch/ternary over a two-or-three-variant union needs exactly that many branches — a leftover catch-all is a sign the union isn't doing its job, or that the old optional-fields shape was only partially migrated.

### Should Have (Important)

- *RULE-101 — Prefer a shared base type intersected with the varying part:* When only some fields vary between variants, factor the common fields into a base type and intersect it with a smaller union of just the differing fields, rather than repeating every shared field inside each variant.

- *RULE-102 — Reach for this before adding a second optional field to an existing type:* The moment a second field becomes optional on a type that already has one, stop and ask whether the two are actually independent or mutually exclusive. If they're related, that's the signal to convert to a union before, not after, a caller manages to construct an invalid combination.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
type Location = {
  name: string
} & (
  | { kind: 'address'; address: string }
  | { kind: 'coordinates'; lat: number; lng: number }
)

function describe(location: Location): string {
  return location.kind === 'address'
    ? location.address
    : `${location.lat}, ${location.lng}`
}
// Exhaustive: exactly two branches for exactly two variants. No "neither" state exists to fall through to.
```

### ❌ Don't Do This

```typescript
type Location = {
  name: string
  address?: string
  lat?: number
  lng?: number
}
// Allows address+lat+lng all set, or all three unset — neither is meaningful,
// but the type happily constructs both.

function describe(location: Location): string {
  return location.address
    ? location.address
    : location.lat
      ? `${location.lat}, ${location.lng}`
      : ''  // ← catch-all for a state that should never have been representable
}
```

## Decision Framework

*When rules conflict:*
1. Type-level guarantees take priority over runtime convenience — a slightly more verbose union is worth it if it removes a class of bug entirely.
2. If migrating an existing optional-fields type to a union would touch many call sites at once, that is a sign the ambiguity was already spreading — do the migration rather than patching one more call site.

*When facing edge cases:*
- A field that is optional because the *caller* hasn't decided yet (e.g. a draft form) is not the same as a field that is optional because it's mutually exclusive with another — only the latter needs a union.
- If a third, genuinely valid "neither" state exists (e.g. "not yet configured"), model it as an explicit third variant, not as the absence of both other fields.

## Related Rules

- design/naming-conventions.md — a name that lies about a type's actual shape often points at the same root problem
- design/code-organisation.md RULE-004 — duplicated conditional checks are the runtime symptom of a shape that should have been a union

---

## TL;DR

*Key Principles:*
- A type should make invalid combinations impossible to construct, not just unlikely
- Validate at construction (the type), not at every use site (a runtime check)

*Critical Rules:*
- Mutually exclusive fields → a discriminated union, never independent optionals (RULE-001)
- Branch on the discriminant, never re-derive the variant from which optional field is set (RULE-002)
- No catch-all branch for a state the union no longer allows (RULE-003)

*Quick Decision Guide:*
Before adding a second optional field to a type: ask whether it's independent of the first, or mutually exclusive with it. Independent → fine as-is. Mutually exclusive → convert to a union now, before a caller constructs the combination the domain forbids.
