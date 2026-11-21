---
name: fanout
description: Run independent subtasks in parallel — one git worktree and one implementation sub-agent per task, each opening its own PR — then independently cross-review and auto-merge eligible PRs.
---

# fanout — safe parallel execution

Use ONLY for subtasks that are parallel-safe (no shared files, no ordering
dependency).

## Procedure
1. Per task, create an isolated worktree:
   `sys_os_shell("git worktree add .worktrees/<task_id> -b polly/<task_id>")`.
   Record the worktree path + branch in the registry
   (`.polly/registry.json`).
2. Dispatch one implementation sub-agent per task, scoped to its worktree:
   `sys_session_send(agent="claude_code"|"codex"|"opencode"|"cursor"|"hermes"|"agy", title="<task_slug>",
   args={purpose: "implement", input: "<task + acceptance contract +
   worktree path>"})`. Use a short task-based title such as `auth-refactor` or
   `fix-sse-error`, never the raw vendor name. State the scope and that it must
   work only inside `.worktrees/<task_id>`. The worker drives the task to green
   and opens its OWN PR for the branch. Every commit the worker authors must
   end with a blank line followed by the exact co-sign trailer as its final
   line — `Co-authored-by: omnigent <noreply@omnigent.ai>`.
   For a long-running `claude_code` or `codex` implementation with an explicit
   completion condition, the `input` may instead be one standalone
   `/goal <condition>` command containing that same task, worktree, acceptance
   contract, green gates, and PR requirement.
   Do not use child goal mode for other workers or non-implementation purposes.
   Record each handle's `conversation_id`
   in the registry. Emit the worktree + `sys_session_send` tool calls in THIS
   turn — never end a turn having only said you will dispatch; the dispatch
   calls and their announcement go in the same turn. Dispatch the whole
   parallel-safe set, THEN (and only then) END YOUR TURN. Do not poll.
3. Each sub-agent runs autonomously and notifies you through the inbox when it
   finishes. Collect its structured result with `sys_read_inbox` and record the
   PR URL in the registry. If the inbox result is empty/unclear, inspect that
   worker conversation with `sys_session_get_history` before deciding what to do
   next.
4. Send each finished task's PR through `cross-review`.
5. After cross-review passes, load the bundled auto-merge skill and apply its
   eligibility checks. The orchestrator alone merges eligible PRs through GitHub.
   Record reviewed, blocked, queued, or merged accurately; a pending or failed
   merge is not completed work. Keep merge operations sequential per target
   branch and revalidate each remaining PR after that branch advances.
6. Remove a finished worktree (`git worktree remove`) only after GitHub confirms
   the PR is merged and there are no open fix-tasks. Preserve worktrees for
   blocked or queued PRs so validation and fixes remain possible.

## Notes
- Respect the per-turn dispatch cap (enforced by policy). More tasks than the
  cap → dispatch in waves (let the running batch finish before dispatching more).
- The human can open any sub-agent in the UI's Subagents panel and read its
  conversation while it runs.
- If a running worker is wrong, runaway, superseded, or no longer useful, call
  `sys_cancel_task` with `task_id` set to the recorded `conversation_id` before
  dispatching a replacement. `claude_code` is hard-stopped; `codex` cancellation
  is best-effort until its runner-side hard-stop exists.
- A sub-agent that returns a dark or failing result: don't re-prompt it in a
  loop — re-dispatch a fresh implementation sub-agent in a clean worktree, or
  escalate to the user.
- Parallel implementation does not imply parallel merging. A preceding merge
  may invalidate another PR's base, gates, or review. Revalidate against the new
  base before merging the next PR; delegate conflict fixes to its implementer.
