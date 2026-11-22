# RoboPolly

Default Polly delegates implementation and independent review, then leaves merging to a human. RoboPolly (`robo-polly`) keeps that workflow and also merges scoped PRs once review, validation, and GitHub's requirements pass.

Modified from `examples/polly` in https://github.com/omnigent-ai/omnigent (Apache-2.0; see `LICENSE` and `NOTICE`).
Changed: `robo-polly/config.yaml`, `robo-polly/skills/{cross-review,fanout}/SKILL.md`; added `robo-polly/skills/auto-merge/SKILL.md`.

## Keep up with upstream Polly

RoboPolly tracks the pristine Polly bundle in the published [Omnigent PyPI
wheel](https://pypi.org/project/omnigent/). `polly-upstream.lock` records the
release and its pristine Git tree. The script uses Git and Python 3 and
downloads the wheel as data.

```sh
scripts/polly-upstream check          # Show the changes in the latest release
scripts/polly-upstream diff           # Show committed RoboPolly customizations
git switch -c update-polly
scripts/polly-upstream update         # Apply the upstream changes in a local commit
scripts/polly-upstream diff           # Review the customizations against the new base
```

Pass a PyPI version such as `0.23.0` after `check` or `update` to select a
specific release. Start an update from a clean worktree. Git applies the
changes between the two pristine releases to the current branch, carrying
RoboPolly's changes forward and updating the pin in the same commit. If Git
reports a conflict, resolve it, stage the resolved paths, and run
`git cherry-pick --continue`. Use `git cherry-pick --abort` to abandon the update.
Review the update commit and remaining customizations before opening a PR.
