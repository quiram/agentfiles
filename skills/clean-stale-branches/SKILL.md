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

## Worktree cleanup

Before branch deletion, check whether any stale branch is currently checked out in a worktree (use `git worktree list --porcelain` to cross-reference stale branch names). A stale branch that's worktree-locked cannot be deleted until its worktree is removed.

For each stale branch with an associated worktree:

1. **Skip if it's the current worktree** — this session's working directory (`pwd`). Never remove a worktree in use.
2. **Skip if the worktree is locked** — `git worktree list` shows the lock flag if another Claude Code session or process holds it. Locked worktrees belong to another session; do not attempt removal.
3. **Check for uncommitted/untracked changes** — run `git status --short` *inside the worktree directory* and confirm it is empty. Same bar as the unpushed-commits check for branches: dirty worktrees are unsafe to remove.
4. **If clean, eligible for removal** — `git worktree remove <path>` is safe.
5. **If dirty or locked, skip and warn** — same treatment as a branch with unpushed commits.

After all eligibility checks, present worktrees to be removed to the user for confirmation before proceeding with any removals.

## Safety rules

Before deleting **anything**:

1. **Show the user the full list first** — branch name, why it's classified stale (which signal matched), and the PR number if applicable. Also list any worktrees to be removed. Wait for explicit confirmation before any deletion or removal.
2. **Remove worktrees before deleting branches.** If a stale branch is checked out in a worktree, remove the worktree first (using `git worktree remove <path>`), then delete the now-unblocked branch.
3. **Never remove a worktree in use** — skip the current worktree (this session's `pwd`). Never remove a locked worktree (another session holds it).
4. **Never remove dirty worktrees** — check `git status --short` inside each worktree; skip and warn if there are uncommitted or untracked changes. Dirty worktrees are unsafe to remove, same as branches with unpushed commits.
5. **Never delete the current branch.** If a branch you'd delete is currently checked out (outside a worktree), skip it and tell the user.
6. **Never delete branches with unpushed commits the remote doesn't have**, unless the branch has a merged PR (the PR proves the work landed via squash, so the local commits being "ahead" is expected). For non-PR branches, check `git log <branch> --not --remotes` — if it has unique commits, skip and warn.
7. **Never use `-D` (force delete) blindly.** Use `git branch -d <branch>` first; if that fails because Git thinks the branch isn't merged (typical for squash-merge), only then use `-D` — and only for branches that matched signal **2** (merged PR). The merged PR is the authorisation to force-delete.
8. **Never touch the remote.** This skill removes local worktrees and deletes local branches only. Remote branch cleanup is GitHub's job (auto-delete on merge) or a separate explicit task.

## Workflow

1. List local branches: `git for-each-ref --format='%(refname:short)' refs/heads/`.
2. For each branch, classify it (stale / not stale / skipped) using the detection strategy.
3. Cross-reference stale branches against `git worktree list --porcelain` to identify which stale branches are checked out in worktrees.
4. For each stale branch with a worktree, check worktree eligibility (not current, not locked, not dirty).
5. Group the output into sections:
   - **Worktrees to remove** — clean, non-locked, non-current worktrees holding stale branches.
   - **Stale (merged PR)** — safe to force-delete once worktrees are removed.
   - **Stale (merged into default / ancestor of default)** — safe to delete with `git branch -d` once worktrees are removed.
   - **Skipped** — current branch, default branch, locked worktrees, dirty worktrees, or branches with unpushed work.
6. Present the lists to the user and ask for confirmation. Do not assume a previous "yes I want to clean up" extends to deletion — that yes was for analysis.
7. On confirmation, remove worktrees first: `git worktree remove <path>` for each eligible worktree.
8. Then delete branches in two passes:
   - `git branch -d <branch>` for stale branches merged into default.
   - `git branch -D <branch>` for stale branches with merged PRs.
9. Report what was removed (worktrees) and deleted (branches) and what remains.

## Output format

When presenting the analysis, use a compact table the user can scan. Example:

```
Worktrees to remove:
  .claude/worktrees/fix-9799        fix/9799-404-lookup          PR #11205 merged 2026-08-31
  .claude/worktrees/chore-forbid    chore/forbid-relative-imports PR #11199 merged 2026-08-31

Stale — squash-merged (will need -D):
  feat/35-course-expiry         PR #71  merged 2026-04-12
  fix/ci-duplicate-runs         PR #58  merged 2026-03-21

Stale — merged into main:
  chore/roadmap-files-reference

Skipped:
  main                          (default branch)
  feat/9798-consolidate-404    (current branch)
  feat/11139-klarna-pay-by-bank (.claude/worktrees/klarna — locked by another session)
  feat/m4.13-error-handling     (has 2 unpushed commits, no merged PR)
```

Then ask: "Remove the 2 worktrees and delete the 3 stale branches above? (y/N)"

## Edge cases

- **`gh` not installed or not authenticated.** Fall back to signals 3 and 4 only, and warn the user that squash-merged branches will not be detected — they should install/auth `gh` for a complete sweep.
- **Branch points at a commit on the default branch already** (e.g. someone reset it). Signal 4 catches this.
- **Detached HEAD.** Don't worry about it — `for-each-ref refs/heads/` only lists named branches.
- **Multiple remotes.** Use `origin` consistently; if the repo uses a different primary remote, the user should run `git symbolic-ref refs/remotes/<remote>/HEAD` against it instead.
- **Worktree is locked.** `git worktree list --porcelain` shows `locked` if another Claude Code session or process holds it. Do not attempt to remove locked worktrees — skip and warn. The lock is released when the other session exits.
- **Worktree directory is missing or prunable.** If a worktree entry in `git worktree list` points to a non-existent path, run `git worktree prune` separately to clean up the stale entry — this skill does not attempt `git worktree remove` on missing worktrees.
- **Worktree has uncommitted/untracked changes.** `git status --short` inside the worktree must be empty. If there are changes, skip and warn the user to commit or stash them first — do not remove dirty worktrees.
- **Current session is in the worktree.** Never remove the worktree in use by this session (`pwd`). Skip and warn.
