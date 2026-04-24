# GitHub Issues

Rules for interacting with GitHub issues in this project.

## Context

*Applies to:* Any task referencing a GitHub issue number, or any time a new issue is created
*Level:* Operational — applies every time an issue is mentioned or created
*Audience:* AI agents working on this project

## Core Principles

1. *Read before acting:* Always fetch the issue before searching branches, writing code, or making assumptions about scope
2. *Issues belong to milestones:* Every issue must be assigned to a milestone; a milestone-less issue has no place in the roadmap

## Rules

### Must Have (Critical)

- *RULE-001:* When the user refers to a GitHub issue (e.g. "issue #32"), run `gh issue view <number> --repo quiram/pancomido-website` immediately — do not guess branch names or scope from the issue number alone
- *RULE-002:* The issue view output includes any linked branch and PR — use those to find the correct branch or PR to check out rather than searching manually
- *RULE-003:* When creating a new issue, always assign it to a milestone:
  1. List milestones: `gh api repos/quiram/pancomido-website/milestones --jq '.[] | {number, title}'`
     (`gh milestone list` does not exist — always use `gh api`)
  2. Create the issue without `--milestone`, then assign it via API:
     `gh api repos/quiram/pancomido-website/issues/<number> --method PATCH --field milestone=<milestone-number>`
     (the `--milestone` flag on `gh issue create` is unreliable — patching via API is the safe path)
  3. If no existing milestone fits, create one first: `gh api repos/quiram/pancomido-website/milestones --method POST --field title='...'`

## Related Rules

- git/commit-messages.md — commit message conventions, including issue references
- meta/task-execution.md — task execution workflow

---

## TL;DR

*Critical Rules:*
- Always `gh issue view <number> --repo quiram/pancomido-website` before doing anything with an issue
- Use the linked branch/PR from the issue output — don't search for branches manually
- Every new issue must have a milestone — list with `gh api`, create without `--milestone`, then patch via `gh api` (RULE-003)
