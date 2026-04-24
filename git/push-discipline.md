# Push Discipline

Rules for when to commit and when to push, to avoid triggering excessive automated PR reviews.

## Context

*Applies to:* All work on branches with open or draft PRs in this project
*Level:* Operational — applies on every push decision
*Audience:* AI agents working on this project

## Background

This project runs a Claude agent to review every PR push. Each review consumes a significant number of tokens. Pushing frequently during a working session triggers multiple reviews and exhausts the token budget quickly.

## Core Principles

1. *Commit often, push once:* Keep local history clean and atomic, but batch all pushes to the end of a session
2. *Push only when told:* Never push unless the user explicitly asks

## Rules

### Must Have (Critical)

- *RULE-001:* Commit locally after each sub-task as normal — commits must remain atomic and self-contained (see git-rules.md)
- *RULE-002:* Do NOT push until the user explicitly says to push — regardless of how many commits have accumulated locally
- *RULE-003:* Never push as part of completing a task or sub-task; wait for explicit instruction

## Related Rules

- git/git-rules.md — atomic commit and branch conventions
- meta/task-execution.md — sub-task commit workflow

---

## TL;DR

*Critical Rules:*
- Commit after every sub-task; push only when the user explicitly says to
