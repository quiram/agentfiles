# Branch Discipline

Rules for ensuring all work happens on a feature branch, never directly on `main`.

## Context

*Applies to:* All work in this repository — issue-driven and ad-hoc alike
*Level:* Operational — applies before any code is written
*Audience:* AI agents and developers starting any change

## Core Principles

1. *Main is always deployable:* No work-in-progress ever lands directly on `main`
2. *Branch first, code second:* The branch is created before the first edit, not retroactively after commits accumulate

## Rules

### Must Have (Critical)

- *RULE-001:* Before making any code change, verify you are on a feature branch (`git branch --show-current`). If on `main`, create and switch to a feature branch immediately — do not edit, stage, or commit on `main`.
- *RULE-002:* Branch names follow `type/<short-description>` (per git-rules.md RULE-101). When the work is tied to a GitHub issue, prefix the description with the issue number: `feat/35-course-expiry`, `fix/42-checkout-error`. For ad-hoc work, omit the number: `chore/tidy-readme`, `refactor/extract-invoice-service`.
- *RULE-003:* If commits have already landed on `main` by mistake, recover by: (a) creating the feature branch from current `HEAD` to preserve the work, (b) resetting `main` back to `origin/main` with `git branch -f main origin/main`. Never push the accidental commits from `main`.

## Related Rules

- agents/ee-llm-toolkit/rules/git-rules.md — branch naming conventions (RULE-101) and "never commit directly to main" (RULE-005)
- git/github-issues.md — using linked branches from existing issues
- git/push-discipline.md — when to push the branch

---

## TL;DR

*Critical Rules:*
- Check `git branch --show-current` before the first edit; if it's `main`, branch first
- Issue work: `type/<number>-<description>`. Ad-hoc work: `type/<description>`
- If you slip and commit on `main`, move the commits to a branch and reset `main` to `origin/main`
