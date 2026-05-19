# Logging Conventions

Rules for how logging is written. Partially overrides the logging rules in `agents/ee-llm-toolkit/rules/code-quality.md`.

## Context

*Applies to:* All logging statements
*Level:* Tactical/Operational — applies whenever a log statement is written or reviewed
*Audience:* All developers and AI agents

## Override Notice

This rule **partially overrides** the logging rules of `agents/ee-llm-toolkit/rules/code-quality.md`, which state:

> only use structured logging, flat structure, key value pairs
> do not use templates or verbose story telling logs

Those rules are appropriate for high-traffic systems with dedicated log aggregation tooling (Datadog, Grafana, Splunk, etc.) where machine-parseability of individual fields enables automated alerting, dashboards, and querying at scale.

For low-traffic projects using basic log viewing (e.g. a cloud provider's built-in log viewer), human-readable messages are more valuable than pure key-value pairs. This rule applies to **low-traffic projects**; high-traffic projects should follow the ee-llm-toolkit rules instead.

Before applying this rule, check the project context to determine which category applies.

## Rules

### Must Have (Critical)

- *RULE-001:* Log statements must be human-readable at a glance. A developer scanning logs to confirm a job fired or debug a failure should understand a log line without decoding it.

- *RULE-002:* Pass structured data as a second argument to `console.log` / `console.error` rather than interpolating it into the message string. This preserves both a readable message and machine-readable context.

- *RULE-003:* Use `console.error` for failures and `console.log` for informational output. Do not introduce a logging library unless one is already in use in the project.

### Should Have (Important)

- *RULE-101:* A short prefix or emoji is acceptable to make log lines easy to spot when scanning. Consistency within a module matters more than a global convention.

- *RULE-102:* Background jobs (cron jobs, queue processors, etc.) must log a summary line on each run so that correct operation can be confirmed by inspection.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
// Readable message + structured data as second argument
console.log('📊 Expiry run complete:', { expired })
console.log('✅ Contact created:', { email, contactUrl })
console.error('❌ Invoice failed:', { paymentIntentId, error: err.message })
```

### ❌ Don't Do This

```typescript
// Interpolated data — no structured context for tooling
console.log(`Expiry run complete, expired: ${expired}`)

// Pure key-value with no readable message — hard to scan
console.log({ event: 'expiry_run_complete', expired })
```

## Related Rules

- agents/ee-llm-toolkit/rules/code-quality.md — base code quality rules; logging section overridden by this file for low-traffic projects

---

## TL;DR

*Applies to:* Low-traffic projects. High-traffic projects use ee-llm-toolkit logging rules instead.

*Critical Rules:*
- Log statements must be human-readable (RULE-001)
- Pass structured data as a second argument, not interpolated into the string (RULE-002)
- Background jobs must emit a summary log on each run (RULE-102)

*Quick Decision Guide:*
Could a developer scanning logs understand this line in 2 seconds? If yes, it's fine. If it reads like a JSON blob or a cryptic key-value pair with no message, make it more readable.
