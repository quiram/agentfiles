# GitHub Issues

Rules for interacting with GitHub issues in this project.

## Context

*Applies to:* Any task referencing a GitHub issue number
*Level:* Operational — applies every time an issue is mentioned
*Audience:* AI agents working on this project

## Core Principles

1. *Read before acting:* Always fetch the issue before searching branches, writing code, or making assumptions about scope

## Rules

### Must Have (Critical)

- *RULE-001:* When the user refers to a GitHub issue (e.g. "issue #32"), run `gh issue view <number> --repo quiram/pancomido-website` immediately — do not guess branch names or scope from the issue number alone
- *RULE-002:* The issue view output includes any linked branch and PR — use those to find the correct branch or PR to check out rather than searching manually

## Related Rules

- git/commit-messages.md — commit message conventions, including issue references
- meta/task-execution.md — task execution workflow

---

## TL;DR

*Critical Rules:*
- Always `gh issue view <number> --repo quiram/pancomido-website` before doing anything with an issue
- Use the linked branch/PR from the issue output — don't search for branches manually
