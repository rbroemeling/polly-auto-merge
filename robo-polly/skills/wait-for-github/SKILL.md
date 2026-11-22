---
name: wait-for-github
description: Wait for GitHub PR checks, reviews, or state changes.
---

# wait-for-github

Use one deadline across retries (30 minutes by default). Read fresh status
with `gh pr view <url> --json state,isDraft,headRefOid` (add `reviews` as
needed) and `gh pr checks <url> --required --json name,bucket,link` for CI.
A suite event or empty check list does not show that required checks passed.

While pending, use `sys_call_async` with `sys_os_shell` for a verified
`github.com` PR URL. Cap `--timeout` at 60 seconds or the time left and set
the shell timeout slightly higher.

```bash
gh event listen --resource 'https://github.com/OWNER/REPO/pull/123' \
  --event com.github.pull_request.check_suite.status.changed.v1 \
  --event com.github.pull_request.head.changed.v1 \
  --event com.github.pull_request.state.changed.v1 \
  --timeout 60s --jq .type
```

Select events that affect the condition, including
`com.github.pull_request.review.submitted.v1` for reviews. Keep one handle
per PR. Recheck after dispatch, then yield and collect its result with
`sys_read_inbox` on wake. Recheck after each event or timeout (exit 8), which
catches missed events. If listening fails or is unavailable, poll every 15
seconds within the same deadline. When the caller needs a fresh event or
refresh, wait one window even if the initial status looks ready.

Return fresh status when the condition is met, the expected head changes,
a blocker appears (including failed or cancelled required checks), or time
runs out. Cancel any outstanding handle with `sys_cancel_async`.
