# RoboPolly maintenance

Maintain `robo-polly/` as a minimal floating diff on top of the pristine Polly bundle shipped in the latest Omnigent wheel published on PyPI. `polly-upstream.lock` records the pinned Omnigent release and its pristine Polly Git tree.

- Run `./polly-upstream check` to review upstream changes since the pin. To track a new release, run `./polly-upstream update` from a clean worktree. It applies the upstream changes and advances the pin in the same commit. Review that commit and drop local changes when upstream provides equivalent behavior.
- Keep RoboPolly-specific changes small and focused. Reuse upstream behavior wherever it meets the requirement, and preserve upstream file layout, naming, and formatting wherever possible.
- Before opening or updating a PR that changes `robo-polly/`, commit the bundle changes and run `./polly-upstream diff`. It compares the committed bundle with the pinned pristine tree. Verify that each remaining difference is needed for RoboPolly.
