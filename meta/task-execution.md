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
- *RULE-002b:* Tests for a piece of functionality must be committed in the same commit as that functionality. A commit that introduces new behaviour without tests, or a commit that only adds tests for previously committed behaviour, violates commit cohesion. The red-green cycle determines the order of writing; the commit captures both together.
- *RULE-003:* Never batch multiple sub-tasks into a single commit.
- *RULE-004:* All changes must be made within `agents/agentfiles/`. Never modify `agents/ee-llm-toolkit/` — it is a read-only submodule.
- *RULE-005:* When running as the GitHub Claude action, do not modify content inside `agents/agentfiles/`. The action has no push access to the agentfiles submodule's remote, so any commit you make there is orphaned: the parent-repo commit that bumps the submodule SHA will reference a non-existent ref, leaving CI unable to init submodules and the PR unmergeable without manual recovery. Instead, leave a comment on the PR or issue describing the proposed agentfiles change — file path, full content, and rationale — so the maintainer can apply it locally and push direct to agentfiles main.

## Related Rules

- agents/ee-llm-toolkit/rules/task-execution.md — base task execution workflow (read-only reference)
- agents/ee-llm-toolkit/rules/git-rules.md — commit message and branching conventions

---

## TL;DR

*Critical Rules:*
- Plan → approval → implement → quality checks → approval → commit — one sub-task at a time
- Quality checks = `npm test` and `npm run quality` with zero failures. Any failing test blocks progress, regardless of apparent cause.
- Tests commit with their functionality — never separated (RULE-002b)
- Never touch `agents/ee-llm-toolkit/` — write rules in `agents/agentfiles/` only
- When running as the GitHub Claude action, never modify `agents/agentfiles/` — leave a PR/issue comment with the proposed change instead (RULE-005)
