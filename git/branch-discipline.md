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

- *RULE-001:* Before making any code change, verify you are on the *right* feature branch for this work — not just any non-`main` branch. Run `git branch --show-current` and check it matches the task. If on `main`, or on a branch for a different task (e.g. on `feat/19-...` while starting issue #30), check whether an appropriate branch already exists (`git branch --list 'feat/30-*'`, `git branch --list '*<keyword>*'`); switch to it if it does, or create one per RULE-002 if it doesn't. The user may explicitly direct otherwise (e.g. "stay on main", "use this branch", "create branch X") — when they do, follow their instruction without re-checking.
- *RULE-002:* Branch names follow `type/<short-description>` (per git-rules.md RULE-101). When the work is tied to a GitHub issue, prefix the description with the issue number: `feat/35-course-expiry`, `fix/42-checkout-error`. For ad-hoc work, omit the number: `chore/tidy-readme`, `refactor/extract-invoice-service`.
- *RULE-003:* If commits have already landed on `main` by mistake, recover by: (a) creating the feature branch from current `HEAD` to preserve the work, (b) resetting `main` back to `origin/main` with `git branch -f main origin/main`. Never push the accidental commits from `main`.
- *RULE-004:* When a governance tweak (rule edit, CLAUDE.md change, small process fix) is discovered or discussed mid-session, treat it like a refactor: small, self-contained tweaks may ride on the current feature branch as additional commits; large governance changes (new rule files, multi-file restructures, anything that warrants its own review and discussion) need their own branch and PR. When in doubt, ask the user.

## Related Rules

- agents/ee-llm-toolkit/rules/git-rules.md — branch naming conventions (RULE-101) and "never commit directly to main" (RULE-005)
- git/github-issues.md — using linked branches from existing issues
- git/push-discipline.md — when to push the branch

---

## TL;DR

*Critical Rules:*
- Check `git branch --show-current` before the first edit; if it's `main` or the wrong branch for this task, switch or branch first
- Look for an existing branch matching the task before creating a new one; user instructions override the check
- Issue work: `type/<number>-<description>`. Ad-hoc work: `type/<description>`
- If you slip and commit on `main`, move the commits to a branch and reset `main` to `origin/main`
- Small governance tweaks discovered mid-session can ride on the current branch; large ones need their own branch and PR (ask if unsure)
