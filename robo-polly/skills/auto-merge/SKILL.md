---
name: auto-merge
description: Check the title and description of a PR that passed cross-review, then merge it once GitHub allows it and confirm the merge.
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
     feedback for it: `--squash`.
   - A PR with several objectives: `--rebase` or `--squash`, by judgment.

   If the repository does not allow the chosen method, do not merge; report
   that as the blocker.

3. Just before each merge attempt, read the current PR metadata with
   `gh pr view <number> --repo <owner/repo> --json baseRefName,baseRefOid,headRefOid,title,body`.
   If the head differs from the reviewed head, return to step 1. Compare the
   title and description with the full PR diff from
   `gh pr diff <number> --repo <owner/repo>`.

   A clear title names the main change. A concise description covers the
   material changes and how they were verified, using the recorded validation
   results. Edit only fields that are stale, unclear, or unnecessarily long.
   Preserve relevant issue links, closing keywords, and required template
   sections.

   Re-read the metadata after the comparison. If it changed while you were
   checking the diff, repeat this step with the latest values. For any edits,
   use `gh pr edit <number> --repo <owner/repo>` with `--title` for a title change
   and `--body-file` for a description change. Confirm the saved values and
   repeat this step if the base or head changed during the edit. If a required
   edit fails, report the blocker and leave the PR open.

4. For a squash merge, write a concise `--subject` and `--body` consistent with
   the checked PR title and description. Merge with an explicit repository and
   the reviewed head pinned:
   `gh pr merge <number> --repo <owner/repo> --rebase --match-head-commit <reviewed-head-sha>`
   or
   `gh pr merge <number> --repo <owner/repo> --squash --subject "<subject>" --body "<body>" --match-head-commit <reviewed-head-sha>`
   Never use `--admin` or otherwise bypass required checks, approvals, or
   branch protections. Do not use `--auto`. If required checks are still
   pending, wait and retry from step 3.
5. Confirm with `gh pr view <number> --repo <owner/repo> --json state,mergeCommit`
   that the state is `MERGED`. Otherwise report the PR as not merged, with
   GitHub's blocker.
