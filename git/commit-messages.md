# Commit Message Conventions

Rules for writing clear, useful commit messages that serve as lasting documentation for the codebase. These rules extend and clarify `agents/ee-llm-toolkit/rules/git-rules.md` — follow both.

## Context

*Applies to:* Every commit in every project under version control
*Level:* Tactical/Operational — applies every time a commit is made
*Audience:* All developers committing code

## Core Principles

1. *Describe the fix, not the bug:* The summary says what was done, not what was wrong
2. *Why over what:* The diff already shows what changed; the message explains the motivation and context
3. *Lasting documentation:* A good commit message is the answer to "why on earth did we do it that way?" six months from now

## Rules

### Must Have (Critical)

- *RULE-001:* Do not copy-paste ticket or issue titles as the commit message — write a summary of what the fix actually does
- *RULE-002:* If the project uses an issue tracker, every commit must reference a work item at the end of the body (e.g. `Fixes #123`, `Refs #123`) — this relaxes `git-rules.md` RULE-004, which is otherwise mandatory

### Should Have (Important)

- *RULE-101:* For any non-trivial change, add a body paragraph (or two) explaining the **motivation**: what was happening before, why it was a problem, and how the approach taken addresses it
- *RULE-102:* Include links to supporting material where relevant — tickets, benchmark results, screenshots, Stack Overflow answers, memory graphs
- *RULE-103:* Trivial changes (typo fixes, documentation tweaks) may use a summary line only — no body required, but the summary must still be specific (not "Update README")

### Could Have (Preferred)

- *RULE-201:* For performance changes, include before/after benchmark numbers in the body

## Patterns & Anti-Patterns

### ✅ Do This

```
feat(auth): only show profile pages for current geo in editor

Previously we were loading profiles for all geos, which confused the
user and made it so they couldn't delete profile pages on other geos.

When the geo dropdown is changed, re-fetch the profile pages so the
list is up to date.

Fixes #988
```

```
fix(ssr): re-enable React server-side rendering

Move render string output directly into the template render call.
By removing it from Koa state we no longer have memory allocation issues.

Results: https://example.com/memory-graph
```

```
docs(linux): add notes about how to build on Linux
```
*(summary-only is fine for trivial changes)*

### ❌ Don't Do This

```
[#988] Cannot delete agent profile in incorrect geo
```
*(copied ticket title — describes the bug, not the fix, no motivation)*

```
fix: remove versioning from deploy-assets scripts
```
*(fine summary, but no motivation — reviewer is left wondering why)*

```
chore: update README.md
```
*(no-op message — what was updated? why?)*

## Decision Framework

*When unsure whether to add a body:*
- If someone reading only the summary might ask "why?" — add a body
- If the diff is entirely self-explanatory — summary only is fine

## Quality Gates

- *Code review focus:* The commit message is a reviewable artefact — reviewers should flag missing motivation or vague summaries just as they would flag bad code
- *History check:* Before pushing, read your commit messages in `git log --oneline` — if any line is unclear out of context, improve it

## Related Rules

- `agents/ee-llm-toolkit/rules/git-rules.md` — Broader git workflow conventions; this file extends it

---

## TL;DR

*Key Principles:*
- The message explains *why*; the diff explains *what*
- Write for the person reading `git log` in six months

*Critical Rules:*
- Never copy-paste ticket titles
- Add a body explaining motivation for any non-trivial change
- Reference a work item if the project uses an issue tracker

*Quick Decision Guide:*
Read your summary aloud as "This commit will ___." If it sounds vague, incomplete, or describes a problem rather than a solution — rewrite it.
