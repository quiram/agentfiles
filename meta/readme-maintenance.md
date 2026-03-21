# README Maintenance

Rules for keeping `README.md` accurate and up to date as the codebase evolves. An outdated README misleads developers and AI agents about the true state of the project.

## Context

*Applies to:* All changes to the project that affect the public interface, structure, or integrations described in `README.md`
*Level:* Operational — applies whenever significant changes are made
*Audience:* Developers and AI agents making changes to the project

## Core Principles

1. *README as source of truth:* The README must reflect the actual state of the project, not its planned or historical state
2. *Update at the point of change:* README updates belong in the same commit as the feature or change that makes them necessary
3. *Status accuracy:* Technologies and integrations must be labelled correctly — never leave something marked *(planned)* once it is live, and never omit a *(planned)* marker for something not yet built

## Rules

### Must Have (Critical)

- *RULE-001:* After any significant change — new integration, new script, structural refactor, dependency added — update `README.md` in the same PR
- *RULE-002:* The tech stack table must accurately reflect what is currently in use; *(planned)* labels must be removed when the technology is live
- *RULE-003:* The scripts table must list all scripts in `package.json` that a developer would need to know about; remove scripts that no longer exist
- *RULE-004:* The project structure section must reflect the actual directory layout; it must not omit major directories (`lib/`, `__tests__/`, etc.)
- *RULE-005:* Integration sections (Stripe, FreeAgent, Resend, etc.) must describe what is currently implemented — remove "*(upcoming)*" markers when a feature ships

### Should Have (Important)

- *RULE-101:* If there is no `README.md`, create one before or alongside the first significant feature

## Exceptions & Waivers

*Valid reasons for exceptions:*
- Pure refactors that change no user-visible behaviour and no project structure — README update is optional
- Trivial dependency version bumps with no API change

## Related Rules

- agents/rules/npm-scripts.md — defines the canonical script names the README must reflect

---

## TL;DR

*Critical Rules:*
- Update `README.md` in the same PR as the change that requires it
- Remove *(planned)* labels when things go live; add them when things are not yet built
- Keep the tech stack, scripts, and project structure sections accurate

*Quick Decision Guide:*
When in doubt: if the README would mislead a developer (or an AI agent) about how the project works right now, update it.
