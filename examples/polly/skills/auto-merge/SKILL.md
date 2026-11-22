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

3. Merge with an explicit repository and the reviewed head pinned:
   `gh pr merge <number> --repo <owner/repo> --rebase --match-head-commit <reviewed-head-sha>`
   or
   `gh pr merge <number> --repo <owner/repo> --squash --subject "<subject>" --body "<body>" --match-head-commit <reviewed-head-sha>`
   Never use `--admin` or otherwise bypass required checks, approvals, or
   branch protections. Do not use `--auto`. If GitHub does not allow the merge
   yet, wait for PR events instead of polling. Start this in the background:
   `gh event listen --resource <pr-url> --event com.github.pull_request.review.submitted.v1,com.github.pull_request.check_suite.status.changed.v1,com.github.pull_request.head.changed.v1,com.github.pull_request.state.changed.v1,com.github.pull_request.merge_conflict.v1`
   Each time it prints `Listening for events.` to stderr, check the PR again,
   because earlier events are not replayed. When it exits after an event,
   return to step 1. An event only prompts this recheck; GitHub's requirements
   still decide. Each check-suite event covers one suite, and commit statuses
   send none, so read all checks with `gh pr checks`. If `gh event listen` is
   unavailable or GitHub refuses access (for example, a 403 because the
   account lacks the event-stream preview), report that as the blocker.
4. Confirm with `gh pr view <number> --repo <owner/repo> --json state,mergeCommit`
   that the state is `MERGED`. Otherwise report the PR as not merged, with
   GitHub's blocker.
