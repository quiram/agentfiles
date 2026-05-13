---
name: clean-stale-branches
description: "Identify and delete local Git branches whose work has already been merged into main, including squash-merged branches that standard tools miss."
---

# Clean Stale Branches

Delete local branches whose work is already on `main`, including ones merged via **squash-and-merge** (which `git branch --merged` does not detect, because the squash commit has a different hash and tree from the branch tip).

## When to use

The user asks to clean up, prune, tidy, or delete stale/merged/old local branches.

## Detection strategy

A local branch is **stale** if any of these hold. Check them in this order, stopping at the first match:

1. **Branch is the default branch or the currently checked-out branch** → never stale, skip.
2. **An associated PR is merged.** Run `gh pr list --state merged --head <branch> --json number,mergedAt --jq '.[0].number // empty' --limit 1`. **Non-empty stdout** means a merged PR exists — the strongest signal, working for squash-merge, rebase-merge, and regular merge equally. Note: `gh pr list ... --json` always returns a JSON array (`[]` when no PR), so a naive `'.[0] | "\(.number) \(.mergedAt)"'` will print the literal `null null` for branches with no PR and produce false positives. The `// empty` filter prevents that trap. Requires `gh` to be available and authenticated against the repo's remote.
3. **Branch is fully merged into the default branch** (regular merge case). `git branch --merged <default>` lists it.
4. **Branch tip is an ancestor of the default branch.** `git merge-base --is-ancestor <branch> <default>` exits 0. Covers fast-forward and already-rebased cases.

If none match, the branch is **not stale** — leave it alone. Do not guess from age, name, or "looks merged-ish".

### Determining the default branch

Use `git symbolic-ref refs/remotes/origin/HEAD --short` (e.g. `origin/main`) and strip the `origin/` prefix. Fall back to `main` if that fails.

## Safety rules

Before deleting **anything**:

1. **Show the user the full list first** — branch name, why it's classified stale (which signal matched), and the PR number if applicable. Wait for explicit confirmation before any deletion.
2. **Never delete the current branch.** If a branch you'd delete is currently checked out, skip it and tell the user.
3. **Never delete branches with unpushed commits the remote doesn't have**, unless the branch has a merged PR (the PR proves the work landed via squash, so the local commits being "ahead" is expected). For non-PR branches, check `git log <branch> --not --remotes` — if it has unique commits, skip and warn.
4. **Never use `-D` (force delete) blindly.** Use `git branch -d <branch>` first; if that fails because Git thinks the branch isn't merged (typical for squash-merge), only then use `-D` — and only for branches that matched signal **2** (merged PR). The merged PR is the authorisation to force-delete.
5. **Never touch the remote.** This skill deletes local branches only. Remote branch cleanup is GitHub's job (auto-delete on merge) or a separate explicit task.

## Workflow

1. List local branches: `git for-each-ref --format='%(refname:short)' refs/heads/`.
2. For each branch, classify it (stale / not stale / skipped) using the detection strategy.
3. Group the output into three sections:
   - **Stale (merged PR)** — safe to force-delete.
   - **Stale (merged into default / ancestor of default)** — safe with `git branch -d`.
   - **Skipped** — current branch, default branch, or has unpushed work.
4. Present the lists to the user and ask for confirmation. Do not assume a previous "yes I want to clean up" extends to deletion — that yes was for analysis.
5. On confirmation, delete in two passes:
   - `git branch -d <branch>` for the second group.
   - `git branch -D <branch>` for the first group (squash-merged).
6. Report what was deleted and what remains.

## Output format

When presenting the analysis, use a compact table the user can scan. Example:

```
Stale — squash-merged (will need -D):
  feat/35-course-expiry         PR #71  merged 2026-04-12
  fix/ci-duplicate-runs         PR #58  merged 2026-03-21

Stale — merged into main:
  chore/roadmap-files-reference

Skipped:
  main                          (default branch)
  claude/issue-87-20260512-0448 (current branch)
  feat/m4.13-error-handling     (has 2 unpushed commits, no merged PR)
```

Then ask: "Delete the 3 stale branches above? (y/N)"

## Edge cases

- **`gh` not installed or not authenticated.** Fall back to signals 3 and 4 only, and warn the user that squash-merged branches will not be detected — they should install/auth `gh` for a complete sweep.
- **Branch points at a commit on the default branch already** (e.g. someone reset it). Signal 4 catches this.
- **Detached HEAD.** Don't worry about it — `for-each-ref refs/heads/` only lists named branches.
- **Multiple remotes.** Use `origin` consistently; if the repo uses a different primary remote, the user should run `git symbolic-ref refs/remotes/<remote>/HEAD` against it instead.
