# Logging Conventions

Rules for how logging is written in this project. Partially overrides the logging rules in `agents/ee-llm-toolkit/rules/code-quality.md`.

## Context

*Applies to:* All logging statements in this codebase
*Level:* Tactical/Operational — applies whenever a log statement is written or reviewed
*Audience:* All developers and AI agents

## Override Notice

This rule **overrides** the logging rules of `agents/ee-llm-toolkit/rules/code-quality.md`, which state:

> only use structured logging, flat structure, key value pairs
> do not use templates or verbose story telling logs

Those rules are appropriate for high-traffic systems with dedicated log aggregation tooling (Datadog, Grafana, etc.) where machine-parseability of individual fields is valuable. This project uses Vercel Observability for occasional human inspection — not automated alerting or dashboards over log fields.

This rule replaces the ee-llm-toolkit logging rules wherever they conflict.

## Rules

### Must Have (Critical)

- *RULE-001:* Log statements must be human-readable at a glance. A developer scanning Vercel logs to confirm a cron job fired or debug a payment failure should be able to understand a log line without decoding it.

- *RULE-002:* Pass structured data as a second argument to `console.log` / `console.error` rather than interpolating it into the message string. This gives both a readable message and machine-readable context.

- *RULE-003:* Use `console.error` for failures and `console.log` for informational output. Do not use a logging library unless one is already in use in the project.

### Should Have (Important)

- *RULE-101:* A short prefix or emoji is acceptable to make log lines easy to spot when scanning — e.g. `📊`, `✅`, `❌`. Consistency within a module matters more than a global convention.

- *RULE-102:* Cron jobs and webhook handlers must log a summary line on each run so that correct operation can be confirmed by inspection.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
// Readable message + structured data as second argument
console.log('📊 Expiry run complete:', { expired })
console.log('✅ FreeAgent contact created:', { email, contactUrl })
console.error('❌ FreeAgent invoice failed:', { paymentIntentId, error: err.message })
```

### ❌ Don't Do This

```typescript
// Interpolated data — harder to parse, no structured context
console.log(`Expiry run complete, expired: ${expired}`)

// Pure key-value with no readable message — hard to scan
console.log({ event: 'expiry_run_complete', expired })
```

## Related Rules

- agents/ee-llm-toolkit/rules/code-quality.md — base code quality rules; logging section overridden by this file

---

## TL;DR

*Critical Rules:*
- Log statements must be human-readable (RULE-001)
- Pass structured data as a second argument, not interpolated into the string (RULE-002)
- Cron jobs and webhook handlers must emit a summary log on each run (RULE-102)

*Quick Decision Guide:*
Could a developer scanning Vercel logs understand this line in 2 seconds? If yes, it's fine. If it reads like a JSON blob or a cryptic key-value pair, make it more readable.
