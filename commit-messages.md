# Commit Message Conventions

Rules for writing clear, useful commit messages that serve as lasting documentation for the codebase.

## Context

*Applies to:* Every commit in every project under version control
*Level:* Tactical/Operational — applies every time a commit is made
*Audience:* All developers committing code

## Core Principles

1. *Single concern:* Each commit addresses one thing; the entire changeset can be described in one simple sentence without omitting anything important
2. *Describe the fix, not the bug:* The summary says what was done, not what was wrong
3. *Why over what:* The diff already shows what changed; the message explains the motivation and context
4. *Lasting documentation:* A good commit message is the answer to "why on earth did we do it that way?" six months from now

## Rules

### Must Have (Critical)

- *RULE-001:* The summary line must use the **imperative mood** — phrase it as "this commit will `<message>`". Write "Add loading states" not "Adding loading states" or "Added loading states"
- *RULE-002:* The summary line must be short (50–72 characters), specific, and cover the entire changeset — if you can't summarise it in one sentence without leaving out important details, the commit is too large
- *RULE-003:* Each commit must target a **single concern** — do not bundle unrelated changes together
- *RULE-004:* Do not copy-paste ticket or issue titles as the commit message — write a summary of what the fix actually does

### Should Have (Important)

- *RULE-101:* For any non-trivial change, add a body paragraph (or two) explaining the **motivation**: what was happening before, why it was a problem, and how the approach taken addresses it
- *RULE-102:* Include links to supporting material where relevant — tickets, benchmark results, screenshots, Stack Overflow answers, memory graphs
- *RULE-103:* Trivial changes (typo fixes, documentation tweaks) may use a summary line only — no body required, but the summary must still be specific (not "Update README")

### Could Have (Preferred)

- *RULE-201:* Reference the ticket or issue number at the end of the body (e.g. `Fixes CNC-988`), not as a substitute for a real summary
- *RULE-202:* For performance changes, include before/after benchmark numbers in the body

## Patterns & Anti-Patterns

### ✅ Do This

```
Only show profile pages for current geo in editor

Previously we were loading profiles for all geos, which confused the
user and made it so they couldn't delete profile pages on other geos.

When the geo dropdown is changed, re-fetch the profile pages so the
list is up to date.

Fixes CNC-988
```

```
Re-enable React server-side rendering

Move render string output directly into the template render call.
By removing it from Koa state we no longer have memory allocation issues.

Results: https://example.com/memory-graph
```

```
Add notes about how to build on Linux
```
*(summary-only is fine for trivial changes)*

### ❌ Don't Do This

```
[CNC-988] Cannot delete agent profile in incorrect geo
```
*(copied ticket title — describes the bug, not the fix, no motivation)*

```
Remove versioning from deploy-assets scripts
```
*(fine summary, but no motivation — reviewer is left wondering why)*

```
Update README.md
```
*(no-op message — what was updated? why?)*

```
Fix bug
WIP
misc changes
```
*(meaningless — tells the reader nothing)*

## Decision Framework

*When the commit feels too large to summarise in one line:*
- Split it into smaller commits, each targeting one concern
- Ask: "could I revert just part of this independently?" — if yes, it should be separate commits

*When unsure whether to add a body:*
- If someone reading only the summary might ask "why?" — add a body
- If the diff is entirely self-explanatory — summary only is fine

## Quality Gates

- *Code review focus:* The commit message is reviewable artefact — reviewers should flag missing motivation or vague summaries just as they would flag bad code
- *History check:* Before pushing, read your commit messages in `git log --oneline` — if any line is unclear out of context, improve it

## Related Rules

- rules/git-rules.md — Broader git workflow conventions

---

## TL;DR

*Key Principles:*
- One commit, one concern — summarisable in a single sentence
- The message explains *why*; the diff explains *what*
- Write for the person reading `git log` in six months

*Critical Rules:*
- Imperative mood: "Add X", "Fix Y", "Remove Z"
- Summary line ≤ 72 chars, specific, covers the whole changeset
- Non-trivial commits need a body explaining motivation
- Never copy-paste ticket titles; never write "fix bug" or "WIP"

*Quick Decision Guide:*
Read your summary aloud as "This commit will ___." If it sounds vague, incomplete, or describes a problem rather than a solution — rewrite it.
