# Push Discipline

Rules for when to commit and when to push, to avoid triggering excessive automated PR reviews.

## Context

*Applies to:* All work on branches with open or draft PRs in this project
*Level:* Operational — applies on every push decision
*Audience:* AI agents working on this project

## Background

This project runs a Claude agent to review every PR push. Each review consumes a significant number of tokens. Pushing once per commit when working on a multi-step goal triggers multiple reviews unnecessarily. The goal is to batch pushes so that each review sees a coherent, complete set of changes.

## Core Principles

1. *Commit often, push in batches:* Keep local history clean and atomic per commit, but group pushes by goal — one push per logical goal, not one push per commit
2. *Push when ready, not on every commit:* Once a goal (or a coherent subset of it) is complete, push. Don't wait for an explicit instruction unless the user has asked you to.
3. *Wait only when told:* If the user explicitly asks you to hold off on pushing, honour that until they say otherwise.

## Rules

### Must Have (Critical)

- *RULE-001:* Commit locally after each sub-task as normal — commits must remain atomic and self-contained (see git-rules.md)
- *RULE-002:* When working toward a multi-step goal that will produce several commits, push **once** at the end of that goal — not after each individual commit. This ensures the automated PR review runs once against the full set of changes rather than multiple times against partial work.
- *RULE-003:* When a goal is complete (or a coherent, self-contained subset of it is done), push without waiting for explicit instruction — unless RULE-004 applies.
- *RULE-004:* If the user explicitly asks you to wait before pushing, do not push until they explicitly say to. This overrides RULE-003 for the remainder of the session (or until the user lifts the hold).

## Related Rules

- git/git-rules.md — atomic commit and branch conventions
- meta/task-execution.md — sub-task commit workflow

---

## TL;DR

*Critical Rules:*
- Commit after every sub-task; push once per goal (not once per commit)
- Push freely when a goal is complete — no explicit instruction needed
- Exception: if the user says to wait, hold all pushes until they say otherwise
