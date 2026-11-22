---
name: auto-merge
description: Merge a PR that passed cross-review once GitHub allows it, then confirm the merge.
---

# auto-merge

Merge only PRs opened for the user's requested implementation tasks, or
existing PRs the user explicitly included. Do not merge if the user asked you
not to.

1. Read fresh PR status with
   `gh pr view <number> --repo <owner/repo> --json state,isDraft,headRefOid,baseRefName,baseRefOid,mergeable,mergeStateStatus`.
   For an already merged PR, go to step 5. For a closed or draft PR, report
   the blocker. On an open, non-draft PR, `mergeable: CONFLICTING` or
   `mergeStateStatus: DIRTY` triggers **Merge conflicts** below, not a stop.
   Check that the current head SHA is the one that passed `cross-review`;
   if it changed, run `cross-review` again first. If mergeability is
   `UNKNOWN`, use `wait-for-github` to await an event or timeout window, then
   reread status from this step within the same deadline; do not merge on
   unknown status or retry in a tight loop. Report a blocker at the deadline.
2. Choose the merge method to keep history clean, with logically coherent
   commits. Never use a merge commit (`--merge`).
   - A PR containing a merge of its base branch: `--squash`, regardless of
     commit count or objectives, with `--subject` and `--body` as below.
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
   branch protections. Do not use `--auto`. After a failed merge attempt,
   reread the status fields from step 1. For an already merged PR, go to
   step 5. For a closed or draft PR, report the blocker. For conflicts, follow
   **Merge conflicts** below, including conflicts that appeared during the
   attempt. If the head changed or mergeability is `UNKNOWN`, go to step 1.
   If required checks are pending, use `wait-for-github` on the reviewed head.
   If it is still current and checks pass on an open, non-draft PR, retry
   from step 1. If GitHub still reports pending, await an event or timeout
   before retrying within the same deadline. Otherwise report the blocker.
5. Confirm with `gh pr view <number> --repo <owner/repo> --json state,mergeCommit`
   that the state is `MERGED`. Otherwise report the PR as not merged, with
   GitHub's blocker.

## Merge conflicts

Conflicts alone are implementation work, not a user blocker. In the registry,
track failed resolution attempts per PR conflict episode (keyed by its initial
base SHA), plus the in-flight worker handle. Until resolution passes the
orchestrator's gates and fresh clean review, keep that episode active: status
refreshes, base/head changes, and worker replacement must not reset its failure
count or dispatch duplicate work. Count each failed attempt once, including a
worker failure, a still-conflicting result, orchestrator gate failures, or a
blocking review. After one failure, retry with concrete corrective instructions,
using existing recovery for dark/failing workers; after two failures, escalate.
Reset only after gates and new review pass, so later base updates start new
episodes without accumulating past successful resolutions.

1. Record a conflict fix-task with the PR URL, current base branch,
   last-reviewed head SHA, original task/acceptance contract, file scope, and
   required gates. Send it to the SAME original implementer conversation via
   `sys_session_send`, reusing its `agent` + `title` (or `session_id`) with
   `purpose: "implement"`, as in `cross-review`. Record the active dispatch and
   follow the usual inbox lifecycle. If the original worker or worktree is
   unavailable, apply the existing fresh-worker recovery rules
   in a clean worktree on the existing PR branch, with the same contract and
   failure count; do not make the conflict alone a user blocker. Record who
   actually authors the resolution, including any replacement implementer.
2. Instruct the worker to fetch the current base and merge it into the PR
   branch, resolving conflicts to preserve both base changes and the task's
   intent within the agreed scope. Do not rebase or force-push. Run all
   applicable full test / lint / typecheck gates, commit with the required
   co-sign trailer, and push normally to the existing PR branch without force
   flags. Return the new head SHA, a per-file resolution summary, and exact
   validation commands/results. If a push is rejected, refresh remote state
   without overwriting it and apply the same failure budget.
3. After the push, read the actual remote head SHA. If it differs from the
   reported SHA, discard validation/review evidence for the reported head;
   identify the actual head's resolution authors and verify it below. If the
   head moves during verification, restart verification on the actual head
   without resetting the episode or failure count.
   Rerun the full deterministic gates on that actual head. On failure, count
   the failed attempt against the episode's budget and, if budget remains,
   send the concrete failing commands/results back to the SAME implementer
   via step 1 (or its recorded recovery replacement); otherwise escalate.
   Do not review or merge a failing head. Handle worker failures and
   blocking reviews through that same bounded fix-task path.
   Follow `cross-review` with a fresh immutable diff snapshot against the
   current base, recording both base/head SHAs. The reviewer's vendor must
   differ from every implementer who actually authored the resolution,
   including a recovered replacement; follow `cross-review`'s availability
   rules if no eligible reviewer exists. The old review cannot carry over to
   the new head. Only after gates pass, the actual head has a clean review,
   and fresh status confirms the conflict resolved, close the episode, reset
   its failure count, and resume at auto-merge step 1;
   a branch containing the base merge commit must use `--squash` with the
   newly reviewed `--match-head-commit`.
4. Escalate genuine unresolved product/behavior or scope decisions with
   concrete conflicting files and options, an episode still unresolved after
   two failed attempts with their results, or a repository that prohibits the
   necessary squash method.
   Leave the PR open and retain all checks, approvals, and branch protections.
