---
name: auto-merge
description: Merge an authorized task PR after independent review and current validation, respecting GitHub protections and confirming the result. Use after cross-review passes in polly-auto-merge.
---

# Auto-merge

The orchestrator performs this procedure; implementers and reviewers never merge.
Apply the user's task scope and any instruction to stop before merging. Selecting
polly-auto-merge authorizes merges of PRs created for requested implementation
tasks in the specified repository and target branch. Review-only, investigation,
and draft-only requests do not authorize merging. Existing PRs must be explicitly
included. Do not infer permission to merge unrelated PRs.

## Establish eligibility

1. Record the exact repository, PR URL/number, intended target branch, current
   base SHA and head SHA in `.polly/registry.json`. Always address GitHub with an
   explicit repository and PR; never rely on the current checkout's PR.
2. Require a successful independent cross-review with zero blocking issues for
   those exact base/head SHAs. Record the immutable diff path, acceptance
   contract, reviewer conversation, actual implementer/reviewer model vendors,
   and validation evidence. Different harness names do not establish different
   vendors. Unknown vendor, missing report, inaccessible diff, or stale evidence
   blocks merging.
3. Independently run the task's required tests, lint, and typecheck at the exact
   PR head against the current target base as appropriate. Record commands,
   revision and outcomes. A passing subset does not replace the agreed gates.
   If any gate fails, return fixes to the implementer and repeat cross-review.
4. Read current GitHub PR metadata, checks, repository rules and review status.
   Verify repository identity and target, open state, non-draft status,
   mergeability, all task-required and repository-required checks and approvals,
   and resolved review conversations. Honor human change requests and any
   explicit hold/do-not-merge instruction. Cross-review does not substitute for
   repository-required GitHub approvals. Unknown or inaccessible status blocks
   merging. Failed, cancelled, pending or missing required checks block merging;
   accept a skipped/neutral result only when the repository and task expressly
   permit it. If no CI is configured, that alone is not a failure: require the
   agreed local validation and affirmative evidence of no required CI checks.
5. Re-read head SHA, base SHA, target and eligibility immediately before merging.
   A changed head or base invalidates local gates and review: repeat validation
   and cross-review for the new pair. Retargeting requires confirming scope.
   Merge PRs targeting the same branch sequentially and refresh remaining PRs
   after each merge. Delegate conflicts and source changes to the implementer.

## Merge and confirm

Use the repository's documented merge method; default to squash when allowed.
If that method is unavailable, report the blocker rather than changing settings.
For an ordinary branch, execute a head-pinned immediate merge, for example:

```text
gh pr merge <number> --repo <owner/repository> --squash --match-head-commit <reviewed-head-sha>
```

Do not use `--admin`, force-push, push directly to the target branch, weaken
protections, approve as the PR author, dismiss reviews, resolve conversations on
others' behalf, or change repository settings to make a merge pass. Do not enable
persistent GitHub auto-merge with `--auto`: a later head could otherwise merge
without Polly reviewing it. This variant achieves automatic merging by completing
its own checks and then requesting a merge immediately.

GitHub merge queues must be respected. When a queue is required, inspect its
rules and current state, use the supported queue submission without bypassing it,
and still pin the head SHA. If the CLI would enable persistent auto-merge rather
than enqueue the eligible reviewed head, do not submit. If reliable head-bound
queue submission is unavailable, leave the PR reviewed and report that blocker.
Queue checks remain GitHub's responsibility; an enqueued PR is not yet merged.

A head-pinned request does not atomically pin the target base. Require current
branch protection or queue validation to enforce up-to-date integration at merge
time when the task requires it. If the environment cannot enforce a required
invariant, report the limitation and leave the PR unmerged.

After the command, query GitHub for the PR's state, merged time and merge commit.
Mark `merged` only on affirmative confirmation and record the PR URL, reviewed
SHAs, method, merge SHA and evidence. Command success alone is insufficient.
Record `queued`, `blocked`, or `reviewed` accurately otherwise. Report failures
with the concrete reason. On an ambiguous timeout or error, read remote state
before retrying; do not blindly issue a second merge request. Do not delete
branches or worktrees while the outcome is unknown or pending.

Pending CI or a queue is a waiting state. Use an available completion notification
or a bounded status watch; report pending state clearly if there is no reliable
completion mechanism. Never claim the implementation is merged while it waits.
A user stop or revoked merge authorization requires cancelling any queued merge
request Polly created and verifying removal; report immediately if removal fails.

The final report includes the PR URL, validation and review outcome, and either
the confirmed merge commit or the precise remaining blocker. This skill grants
no deployment, release, or unrelated-PR authority.
