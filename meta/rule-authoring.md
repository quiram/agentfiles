# Rule Authoring Conventions

Rules for how LLM-facing rules are written and where they live. Ensures all rules in this project are consistent, discoverable, and follow the established template.

## Context

*Applies to:* Any new rule file created for this project
*Level:* Operational — applies every time a rule is authored or updated
*Audience:* Developers and AI agents creating or modifying rules

## Core Principles

1. *Template-driven:* All rules derive their structure from the shared template, ensuring scannable, consistent documents
2. *Centralised location:* Rules live in one known place so agents and developers can always find them
3. *Registered in CLAUDE.md:* A rule that isn't referenced is a rule that won't be applied

## Rules

### Must Have (Critical)

- *RULE-001:* All new rule files must follow the [rule template](https://github.com/EqualExperts/llm-rules/blob/main/prompts/00-rules-template.md) — use its section order, heading names, and RULE-XXX numbering convention
- *RULE-002:* All rule files must be placed under `agents/rules/` in the appropriate subfolder (`git/`, `node/`, `meta/`, etc.) — not in `agents/ee-llm-toolkit/rules/` or anywhere else in the project
- *RULE-003:* Every rule file must be registered in `CLAUDE.md` using the `@agents/rules/<subfolder>/<filename>.md` syntax before it is considered active
- *RULE-004:* Omit template sections that are not relevant — do not include empty or placeholder sections

### Should Have (Important)

- *RULE-101:* Keep each rule file under 200 lines to preserve context window efficiency
- *RULE-102:* File names should be lowercase kebab-case and reflect the rule's intent: `readme-maintenance.md`, `alias-imports.md`
- *RULE-103:* When a rule supersedes or replaces a rule defined elsewhere (e.g. a global `CLAUDE.md`), note that in the file and remove the original

## Patterns & Anti-Patterns

### ✅ Do This

```
agents/rules/meta/readme-maintenance.md   ← correct location
CLAUDE.md includes: @agents/rules/meta/readme-maintenance.md
```

### ❌ Don't Do This

```
docs/rules/readme-maintenance.md                           ← wrong location
agents/rules/meta/readme-maintenance.md (not in CLAUDE.md) ← invisible to agents
```

## Related Rules

- [00-rules-template.md](https://github.com/EqualExperts/llm-rules/blob/main/prompts/00-rules-template.md) — the template all rules must follow

---

## TL;DR

*Critical Rules:*
- Use the [rule template](https://github.com/EqualExperts/llm-rules/blob/main/prompts/00-rules-template.md)
- Place the file in the appropriate subfolder under `agents/rules/`
- Register it in `CLAUDE.md` with `@agents/rules/<subfolder>/<filename>.md`

*Quick Decision Guide:*
When in doubt: template → `agents/rules/<subfolder>/` → `CLAUDE.md`. A rule that isn't in all three places doesn't exist.
