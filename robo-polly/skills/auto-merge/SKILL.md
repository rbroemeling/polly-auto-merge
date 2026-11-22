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
2. Choose the merge method to keep history clean, with logically coherent
   commits. Never use a merge commit (`--merge`).
   - A PR with exactly one commit: `--rebase`.
   - A PR with several commits for one objective, including fixes and review
     feedback for it: `--squash`, with `--subject` and `--body` concisely
     summarizing the changed functionality, consistent with the PR title and
     description.
   - A PR with several objectives: `--rebase` or `--squash`, by judgment.

   If the repository does not allow the chosen method, do not merge; report
   that as the blocker.

3. Before each merge attempt, check that the current title and description
   clearly and concisely reflect the full PR diff. Use `gh pr edit` to update
   only the fields that need it, with `--body-file` for a description edit.
   Preserve issue links, closing keywords, and required template sections. Use
   recorded validation results for any test claims. If an edit fails, report
   the blocker and leave the PR open. Base any squash commit message on the
   checked title and description.

4. Merge with an explicit repository and the reviewed head pinned:
   `gh pr merge <number> --repo <owner/repo> --rebase --match-head-commit <reviewed-head-sha>`
   or
   `gh pr merge <number> --repo <owner/repo> --squash --subject "<subject>" --body "<body>" --match-head-commit <reviewed-head-sha>`
   Never use `--admin` or otherwise bypass required checks, approvals, or
   branch protections. Do not use `--auto`.
   - If GitHub is waiting on required checks, load `wait-for-github` with the
     PR URL, reviewed head SHA, and the condition that required checks pass.
     End the wait early on failing or cancelled required checks, or if the PR
     ceases to be open and ready for review. Use the check suite, head, and
     state events, then decide whether to retry from the fresh status.
   - If the head changed and the PR is still open and ready for review, go
     back to step 1. If the checks passed and the reviewed head is still
     current on an open PR ready for review, return to step 3 before retrying
     the same pinned merge.
     If that retry still reports pending checks, keep the original deadline
     and wait for an event or timed refresh before another attempt, even if
     the check list already shows green.
   - If the PR was already merged, go to step 5. If it closed, became a draft,
     has failing required checks or another merge blocker, or the wait expired
     with checks pending, report the blocker and keep the PR unmerged.
5. Confirm with `gh pr view <number> --repo <owner/repo> --json state,mergeCommit`
   that the state is `MERGED`. Otherwise report the PR as not merged, with
   GitHub's blocker.
