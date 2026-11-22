# RoboPolly

Default Polly agents delegate implementation and independent review, then leave merging to a human. RoboPolly (`robo-polly`) agents keep that workflow and also merge scoped PRs once review, validation, and GitHub's requirements pass.

RoboPolly is meant to be the thinnest possible changeset on top of the default Polly agent in https://github.com/omnigent-ai/omnigent (Apache-2.0; see `LICENSE` and `NOTICE`).

## Installing on Local Omnigent

```sh
./robopolly-install
```

The script fetches `main` from `origin` and installs the committed `robo-polly/`
bundle on the local Omnigent server. Run it again to update the installed agent.

## Tracking Upstream Polly

RoboPolly tracks the pristine Polly bundle in the published [Omnigent PyPI
wheel](https://pypi.org/project/omnigent/). `polly-upstream.lock` records the
release and its pristine Git tree. The script uses Git and Python 3 and
downloads the wheel as data.

```sh
./polly-upstream check          # Show the changes in the latest release
./polly-upstream diff           # Show committed RoboPolly customizations
git switch -c update-polly
./polly-upstream update         # Apply the upstream changes in a local commit
./polly-upstream diff           # Review the customizations against the new base
```

Pass a PyPI version such as `0.23.0` after `check` or `update` to select a
specific release. Start an update from a clean worktree. Git applies the
changes between the two pristine releases to the current branch, carrying
RoboPolly's changes forward and updating the pin in the same commit. If Git
reports a conflict, resolve it, stage the resolved paths, and run
`git cherry-pick --continue`. Use `git cherry-pick --abort` to abandon the update.
Review the update commit and remaining customizations before opening a PR.
