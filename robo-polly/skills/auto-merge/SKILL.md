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

4. Submit the reviewed head to GitHub. Report pending required GitHub
   approvals as blockers.
   - On `github.com`, read the PR's `isMergeQueueEnabled` and
     `mergeQueueEntry { id }` with `gh api graphql`. Report a blocker if the
     queue requirement cannot be verified. If a queue is required, verify
     that `repository.mergeQueue(branch: <baseRefName>).configuration.mergeMethod`
     matches the method chosen in step 2. Report a blocker if the method
     differs or cannot be verified. If the PR is already queued, go to step 5.
     Otherwise use the `enqueuePullRequest` GraphQL mutation with the PR node
     ID as `pullRequestId` and the reviewed head SHA as `expectedHeadOid`.
     Request `mergeQueueEntry { id }` in the response and go to step 5 on
     successful enqueue. `gh pr merge` can implicitly enable auto-merge on a
     queue-required branch.
   - For a direct merge, use
     `gh pr merge <number> --repo <owner/repo> --rebase --match-head-commit <reviewed-head-sha>`
     or
     `gh pr merge <number> --repo <owner/repo> --squash --subject "<subject>" --body "<body>" --match-head-commit <reviewed-head-sha>`.

   Never use `--admin` or otherwise bypass required checks, approvals, or
   branch protections. Do not enable GitHub auto-merge. If the submission
   result is unclear, refresh PR state and queue membership before retrying.
   If GitHub reports that the PR is merged or queued, go to step 5.
   If required checks are pending, use `wait-for-github` on the reviewed head.
   If it is still current and checks pass on an open, non-draft PR, retry from
   step 3. If GitHub still reports pending, await an event or timeout before
   retrying within the same deadline.
   If the head changed on an open, non-draft PR, go to step 1. Otherwise
   report the blocker.
5. Confirm a direct merge with
   `gh pr view <number> --repo <owner/repo> --json state,isDraft,headRefOid,mergeCommit`.
   For a queued PR, use `wait-for-github` on the reviewed head with PR state
   and head change events and the same deadline. On each refresh read
   `state`, `isDraft`, `headRefOid`, `mergeCommit { oid }`, and
   `mergeQueueEntry { id }` together with `gh api graphql`. Queue checks can
   run on a merge group commit, so use PR state to confirm the merge.
   - Confirm success when GitHub reports `MERGED` for the reviewed head. If
     GitHub merged a different head, report the mismatch.
   - If the head changed on an open, non-draft PR, go to step 1.
   - If the PR is still open and non-draft and GitHub removed the queue entry,
     report the removal and any reason GitHub gives. If the deadline expires
     and the entry is still present, report that the PR is still queued.
   - Report any other blocker GitHub gives, including closure or draft status.
