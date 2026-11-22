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
   branch protections. Do not use `--auto`. If required checks are pending,
   use `wait-for-github` on the reviewed head. If it is still current and
   checks pass on an open, non-draft PR, retry from step 3. If GitHub still
   reports pending, await an event or timeout before retrying within the same
   deadline. For an already merged PR, go to step 5. If the head changed on
   an open, non-draft PR, go to step 1. Otherwise report the blocker.
5. Confirm with `gh pr view <number> --repo <owner/repo> --json state,mergeCommit`
   that the state is `MERGED`. Otherwise report the PR as not merged, with
   GitHub's blocker.
