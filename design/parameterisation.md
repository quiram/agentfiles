# Parameterisation Rules

Rules for deciding when a value should be a parameter (or a default) versus being hardcoded. Introducing unnecessary parameters or defaults adds surface area without adding value — keep interfaces as narrow as reality requires.

## Context

*Applies to:* All code — functions, components, classes, configuration
*Level:* Tactical/Operational — applies whenever adding or reviewing parameters and defaults
*Audience:* All developers

## Core Principles

1. *Earn your parameter:* A parameter is justified by actual variability, not imagined future variability
2. *Earn your default:* A default value is justified by frequency — the default path must be taken significantly more often than the override
3. *Push decisions outward:* When a value varies by caller, let the caller supply it; don't reconstruct it inside the callee from raw ingredients

## Rules

### Must Have (Critical)

- *RULE-001:* Only introduce a parameter when real variability exists across callers, or when variability is imminent and concrete. If every caller passes the same value, hardcode it.
- *RULE-002:* Only use a default parameter value when the default is applied significantly more often than the override. If callers are split roughly evenly between the default and an override, there is no meaningful default — make the parameter required and let each caller be explicit.

### Should Have (Important)

- *RULE-101:* When a default exists to generate a value from other parameters (e.g. computing a label from a price and currency), ask whether the generation logic belongs in the callee at all. If callers are already computing similar values, move the generation to the caller and accept the final value directly.
- *RULE-102:* When considering whether to parameterise for future flexibility, prefer to hardcode now and refactor when the variability materialises. Speculative parameters add complexity before they add value. In ambiguous cases, ask the author before introducing the parameter.

## Patterns & Anti-Patterns

### ✅ Do This

```tsx
// Caller constructs the label — the component just renders it
<BuyNowButton slug={course.slug} label={`Buy Now — £${course.price}`} />
<BuyNowButton slug={course.slug} label="Buy Now" size="sm" />

// Currency is always £ — hardcode it
function formatPrice(price: number): string {
  return `£${price}`
}
```

### ❌ Don't Do This

```tsx
// Component reconstructs the label from raw ingredients — why?
// "Buy Now — £246" is computed inside when the caller could just pass it
<BuyNowButton slug={course.slug} price={246} currency="£" />
<BuyNowButton slug={course.slug} price={246} currency="£" label="Buy Now" size="sm" />

// Currency is always £ — no need for a variable
function formatPrice(price: number, currency: string): string {
  return `${currency}${price}`
}
```

## Decision Framework

*When rules conflict:*
1. Favour fewer parameters — a narrower interface is easier to use correctly and harder to misuse
2. If variability is genuinely uncertain, ask the author rather than assuming it will arise

*When facing edge cases:*
- If a parameter is added speculatively and remains unused at its non-default value after a significant period, remove it
- If two callers both use the "default" and zero use an override, delete the default and make the parameter required — the name "default" has become a misnomer

## Related Rules

- rules/clean-code.md — General principles of simplicity and intent-revealing interfaces
- rules/solid-principles.md — Interface Segregation: don't expose what callers don't need

---

## TL;DR

*Key Principles:*
- Parameters earn their place through actual variability; defaults earn their place through frequency
- When in doubt, hardcode it — refactor when reality changes

*Critical Rules:*
- Don't add a parameter unless at least two callers need different values (RULE-001)
- Don't add a default unless the default path is taken significantly more often than the override (RULE-002)

*Quick Decision Guide:*
Before adding a parameter, ask: do two or more callers actually need different values right now? Before adding a default, ask: is the default taken far more often than the override? If the answer to either is "no", don't add it.
