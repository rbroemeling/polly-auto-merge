# RoboPolly

Default Polly delegates implementation and independent review, then leaves merging to a human. RoboPolly (`robo-polly`) keeps that workflow and also merges scoped PRs once review, validation, and GitHub's requirements pass.

Modified from `examples/polly` in https://github.com/omnigent-ai/omnigent (Apache-2.0; see `LICENSE` and `NOTICE`).
Changed: `robo-polly/config.yaml`, `robo-polly/skills/{cross-review,fanout}/SKILL.md`; added `robo-polly/skills/auto-merge/SKILL.md`.
