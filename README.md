# RoboPolly

Default Polly delegates implementation and independent review, then leaves merging to a human. RoboPolly (`robo-polly`) keeps that workflow and also merges scoped PRs once review, validation, and GitHub's requirements pass.

When required checks are pending, RoboPolly listens for GitHub PR events and refreshes the PR's status before retrying. It polls if the event preview is unavailable.

Modified from `examples/polly` in https://github.com/omnigent-ai/omnigent (Apache-2.0; see `LICENSE` and `NOTICE`).
RoboPolly updates `robo-polly/config.yaml` and the `cross-review` and `fanout` skills, and adds the `auto-merge` and `wait-for-github` skills.
