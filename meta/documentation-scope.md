# Documentation Scope

Rules for what project documentation is allowed to describe. Keeps context documents a trustworthy account of the present rather than a mixture of history, plans and wishful thinking.

## Context

*Applies to:* All project documentation — context documents, references, READMEs
*Level:* Operational — applies every time a document is written or edited
*Audience:* Developers and AI agents maintaining project documentation

## Core Principles

1. *Documentation describes the present:* A reader must be able to trust that everything a document states is true of the project right now
2. *The tracker owns the future:* Work that has been agreed but not built belongs in the task tracker, where its status is visible and maintained

## Rules

### Must Have (Critical)

- *RULE-001 — Describe present state only:* Project documentation describes what the project does today. Functionality that has been agreed but not yet implemented does not belong in it, however certain the decision. It enters the documentation at the moment it ships, as part of the work that ships it.

- *RULE-002 — Documentation is not task tracking:* Do not reference tickets, issues or milestones as pending work, and do not maintain lists of planned features. A document that tracks work competes with the tracker and goes stale the moment the tracker moves on. Referencing an issue as the *source* of a recorded decision is acceptable; listing it as outstanding work is not.

- *RULE-003 — Historical state is legitimate where it explains the present:* Describing a system that still exists (a legacy site being migrated from) or a constraint that still binds ("we don't use X because it cannot Y") is present-tense fact and belongs in the documentation. Describing a state the project has left behind, purely as history, does not — that is what version control is for.

## Patterns & Anti-Patterns

### ✅ Do This

```markdown
Private lessons are sold in blocks of 10 and are not purchasable online;
every enquiry routes to the contact form and Bea invoices by hand.

The legacy site sells the same blocks online through WooCommerce at
£450 for 10 lessons.
```

### ❌ Don't Do This

```markdown
Private lessons are not yet purchasable online — a Stripe checkout for
trial lessons is planned (see #9732, milestone 7).

Previously private lessons were sold through WooCommerce, but we removed
this during the September migration.
```

## Decision Framework

*When unsure whether something belongs:*
- Ask whether the statement is true of the project as it stands today. If it needs "yet", "will", "planned" or a ticket number to make sense, it belongs in the tracker.
- Ask whether a newcomer acting on the document alone would do the right thing. A plan read as current state produces wrong work.

## Related Rules

- meta/readme-maintenance.md — keeping `README.md` current as the project changes
- meta/rule-authoring.md — conventions every rule file follows
- git/github-issues.md — the issue body is the specification for work not yet done

---

## TL;DR

*Critical Rules:*
- Documentation describes what the project does today; agreed-but-unbuilt work lives in the tracker until it ships (RULE-001)
- No ticket, issue or milestone references as pending work, and no plan lists (RULE-002)
- Describing a system that still exists, or a constraint that still binds, is present-tense fact and belongs (RULE-003)

*Quick Decision Guide:*
If the sentence needs "yet", "will", "planned", or a ticket number to make sense, it does not belong in the documentation.
