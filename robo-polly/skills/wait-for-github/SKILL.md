---
name: wait-for-github
description: Wait for checks, a review, or a state change on a GitHub pull request, then verify the current PR status.
---

# wait-for-github

1. Identify the PR by URL, the condition to wait for, and the expected head
   SHA if the caller supplied one. Use the caller's deadline, or allow 30
   minutes from the start of this wait. Reuse that deadline across reconnects
   and retries. Take an initial snapshot and stop if it already establishes an
   outcome. If the caller needs a fresh change or timed refresh, complete at
   least one wait window before treating the existing state as success.
   - For PR state and head, use `gh pr view <pr-url> --json state,isDraft,headRefOid,mergeStateStatus,reviewDecision`.
   - For required checks, use `gh pr checks <pr-url> --required --json name,bucket,link`.
     Inspect the buckets. An empty or unavailable result does not establish
     that required checks have passed if GitHub still reports them pending.
   - For a review, include `reviews` in the PR view and check the requested
     reviewer and decision against the relevant head when required.
2. On `github.com`, dispatch `sys_os_shell` with `sys_call_async`. Set `args`
   to a JSON string containing `command` and a shell `timeout` in seconds.
   Let `gh` exit on the first event, and cap its timeout at 60 seconds or
   the remaining time, whichever is shorter. Give the shell a few extra
   seconds so `gh` can return its own exit code. For a required-check wait on
   a verified PR URL, the command is:

   ```bash
   gh event listen --resource 'https://github.com/OWNER/REPO/pull/123' \
     --event com.github.pull_request.check_suite.status.changed.v1 \
     --event com.github.pull_request.head.changed.v1 \
     --event com.github.pull_request.state.changed.v1 \
     --timeout 60s --jq .type
   ```

   For review waits, use `com.github.pull_request.review.submitted.v1` in
   place of the check suite event. For state waits, listen for the state event.
   Include head and state events when a head change or PR closure would make
   the caller's condition obsolete.
3. Record the handle, PR, condition, expected head, and deadline so you can
   resume across turns. Keep one listener in flight per PR. Read GitHub status
   again after dispatch. If the wait is still active, end the turn when
   there is no other work and collect the handle's result with `sys_read_inbox`
   when Omnigent wakes you. Cancel the handle with `sys_cancel_async` if the
   wait ends while the listener is still in flight.
4. Read fresh GitHub status on event delivery or exit, including exit code 8
   (listener timeout). The timeout read catches changes from before attachment
   or during a disconnect. A check suite event covers one suite. Use the
   full required-check and PR state to evaluate the caller's condition. Stop
   if the expected head changes or the PR state rules out the condition. If
   the condition is still pending when the deadline passes, return the latest
   pending status. Otherwise, keep listening after an event or a timeout
   while the condition is pending.
5. If the listener is unavailable or fails, fall back to fresh snapshots about
   every 15 seconds, using the same deadline. Use a bounded background shell
   sleep to yield between snapshots when async calls are available. If GitHub
   status itself cannot be read, report the error.
6. Return the latest status and outcome to the calling skill or report them
   to the user for a standalone wait. The caller decides what action to take.
