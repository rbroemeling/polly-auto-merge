---
name: auto-merge
description: Merge a PR that passed cross-review once GitHub allows it, then confirm the merge.
---

# auto-merge

Merge only PRs opened for the user's requested implementation tasks, or
existing PRs the user explicitly included. Do not merge if the user asked you
not to.

1. Check that the PR's current head SHA is the one that passed `cross-review`.
   If it changed, run `cross-review` again first.
2. Merge with an explicit repository and the reviewed head pinned:
   `gh pr merge <number> --repo <owner/repo> --squash --match-head-commit <reviewed-head-sha>`
   (use the repository's allowed merge method if squash is not allowed).
   Never use `--admin` or otherwise bypass required checks, approvals, or
   branch protections. Do not use `--auto`. If required checks are still
   pending, wait and retry.
3. Confirm with `gh pr view <number> --repo <owner/repo> --json state,mergeCommit`
   that the state is `MERGED`. Otherwise report the PR as not merged, with
   GitHub's blocker.
