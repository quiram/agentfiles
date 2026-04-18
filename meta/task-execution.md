# Task Execution Conventions

Rules for how implementation tasks are planned and executed in this project.
Extends and overrides `agents/ee-llm-toolkit/rules/task-execution.md`.

## Context

*Applies to:* All implementation work in this repository
*Level:* Operational — applies every time a task is started
*Audience:* AI agents and developers executing tasks

## Core Principles

1. *Plan before executing:* No code is written until a plan is approved
2. *One sub-task, one commit:* Each logical change is implemented, reviewed, and committed before the next begins

## Rules

### Must Have (Critical)

- *RULE-001:* Before writing any code, read the relevant source files and produce a numbered implementation plan listing each sub-task with the files affected and the change to be made. **Wait for explicit approval before proceeding.**
- *RULE-002:* Implement one sub-task at a time. After each sub-task: run quality checks → request review → wait for approval → commit → then move to the next.
- *RULE-002a:* Quality checks means `npm test` and `npm run quality` both pass with zero failures. If any test fails — regardless of whether the failure appears pre-existing or unrelated to the current change — stop and investigate before proceeding. Never dismiss a failing test as "not my fault".
- *RULE-003:* Never batch multiple sub-tasks into a single commit.
- *RULE-004:* All changes must be made within `agents/agentfiles/`. Never modify `agents/ee-llm-toolkit/` — it is a read-only submodule.

## Related Rules

- agents/ee-llm-toolkit/rules/task-execution.md — base task execution workflow (read-only reference)
- agents/ee-llm-toolkit/rules/git-rules.md — commit message and branching conventions

---

## TL;DR

*Critical Rules:*
- Plan → approval → implement → quality checks → approval → commit — one sub-task at a time
- Quality checks = `npm test` and `npm run quality` with zero failures. Any failing test blocks progress, regardless of apparent cause.
- Never touch `agents/ee-llm-toolkit/` — write rules in `agents/agentfiles/` only
