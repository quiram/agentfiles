# .gitignore Conventions

Rules for writing and maintaining `.gitignore` files that are clear, predictable, and easy to reason about.

## Context

*Applies to:* All projects using `.gitignore` files
*Level:* Tactical/Operational — applies when adding or modifying ignore rules
*Audience:* All developers maintaining `.gitignore` files

## Core Principles

1. *Explicit over implicit:* Ignore rules should say exactly what is ignored — no surprises
2. *No exceptions:* Avoid negation patterns (`!path`); they are easy to miss and hard to reason about
3. *Specific over broad:* Prefer listing specific paths to ignore over blanket directory ignores when the directory contains files that should be tracked

## Rules

### Must Have (Critical)

- *RULE-001:* Do not use negation patterns (`!path`) to un-ignore files within an ignored directory; instead, replace the broad ignore with specific entries for each path that should be ignored
- *RULE-002:* When a directory contains both tracked and untracked files, list the untracked paths individually rather than ignoring the whole directory

### Should Have (Important)

- *RULE-101:* Group ignore rules by concern with a comment header (dependencies, build output, local config, IDE, etc.)
- *RULE-102:* Prefer ignoring specific file names or patterns over entire directories when only some contents should be excluded

## Patterns & Anti-Patterns

### ✅ Do This

```gitignore
# Claude local settings
.claude/settings.json
.claude/settings.local.json
```

### ❌ Don't Do This

```gitignore
# Claude local settings
.claude/
!.claude/skills/
```

## Related Rules

- git/commit-messages.md — Commit message conventions for this project

---

## TL;DR

*Critical Rules:*
- Never use `!negation` patterns — replace the broad ignore with specific entries instead
- If a directory has both tracked and untracked content, list the untracked paths explicitly
