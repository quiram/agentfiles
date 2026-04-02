# agentfiles

A collection of reusable rules and instructions for AI coding agents, organised by theme.

## Structure

```
git/
  commit-messages.md      # commit message craft — motivation, body, tracker references
  gitignore.md            # specific ignore entries over blanket ignores with negation
node/
  alias-imports.md        # always use @/ alias imports, never relative paths
  npm-quality.md          # expose lint + type-check via a single `npm run quality` command
  npm-scripts.md          # npm script naming conventions — npm test runs everything
  test-organisation.md    # unit/integration/component test folder structure
meta/
  readme-maintenance.md   # keep README.md accurate as the codebase evolves
  rule-authoring.md       # how to write and register new rules
skills/
  retro/SKILL.md          # /retro — session retrospective, skill improvement, automation discovery
                          # sourced from https://github.com/damacus/agent-skills (MIT licence)
```

These rules extend the broader rule set in [EqualExperts/llm-rules](https://github.com/EqualExperts/llm-rules), which covers hexagonal architecture, DDD, clean code, testing principles, and more. The two repos are designed to be used together but work independently.

---

## Usage

> **Path note:** The examples below assume this repo is added as a submodule at `agentfiles/`. If you check it out elsewhere, adjust all paths accordingly.


### Claude Code

Add this repo as a git submodule and reference individual rules in `CLAUDE.md` using `@` imports. Claude Code will load each file into context automatically.

```bash
git submodule add https://github.com/quiram/agentfiles.git agentfiles
```

```markdown
# CLAUDE.md
@agentfiles/git/commit-messages.md
@agentfiles/node/alias-imports.md
@agentfiles/node/npm-scripts.md
@agentfiles/node/npm-quality.md
@agentfiles/node/test-organisation.md
@agentfiles/meta/readme-maintenance.md
@agentfiles/meta/rule-authoring.md
```

Only reference the rules relevant to your project — omit `node/` rules for non-Node projects, for example.

---

### GitHub Copilot

Copilot reads `.github/copilot-instructions.md`. Reference the rule files directly by path — Copilot will read them from the repository.

```markdown
# .github/copilot-instructions.md

Follow the rules defined in the following files:

- agentfiles/git/commit-messages.md
- agentfiles/node/alias-imports.md
- agentfiles/node/npm-scripts.md
- agentfiles/node/npm-quality.md
- agentfiles/node/test-organisation.md
- agentfiles/meta/readme-maintenance.md
```

This applies to both Copilot Chat in VS Code and the Copilot coding agent (automated PRs).

---

### Cursor

Add this repo as a submodule (or copy the files) into `.cursor/rules/`. Cursor reads all files in that directory automatically.

```bash
# Option A: submodule
git submodule add https://github.com/quiram/agentfiles.git .cursor/rules/agentfiles

# Option B: copy only what you need
cp agentfiles/git/commit-messages.md .cursor/rules/
cp agentfiles/node/alias-imports.md .cursor/rules/
```

Cursor supports `.md` and `.mdc` files. `.mdc` files accept frontmatter to control when a rule applies:

```markdown
---
description: "Alias import conventions"
globs: ["**/*.ts", "**/*.tsx"]
alwaysApply: true
---

(content of alias-imports.md here)
```
