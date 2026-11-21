# polly-auto-merge

A variant of [Omnigent](https://github.com/omnigent-ai/omnigent)'s Polly
multi-agent coding orchestrator that merges PRs itself once they pass an
independent different-vendor review, the agreed validation gates, and every
GitHub-required check, approval, and protection. See
[`examples/polly/skills/auto-merge/SKILL.md`](examples/polly/skills/auto-merge/SKILL.md)
for the exact merge conditions.

## Install

Requires an installed and configured Omnigent (`omnigent --version`).

```sh
git clone https://github.com/rbroemeling/polly-auto-merge.git
cd polly-auto-merge
omnigent agent add ./examples/polly
```

This installs the agent as `polly-auto-merge`, next to the built-in `polly`,
so it appears in the new-session picker. Re-running `omnigent agent add`
replaces your installed copy in place. To try it without installing:

```sh
omnigent run ./examples/polly
```

## Provenance and license

The first commit on `main` is an unmodified copy of `examples/polly/` from
[omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent) at commit
`33f4416a108d2cacff23655ef8c667c15f396045` (`main`, 2026-11-21), together with
that repository's `LICENSE` and `NOTICE`. The auto-merge changes are applied on
top of it and can be inspected with `git diff <first-commit> -- examples/polly`.

Modified files: `examples/polly/config.yaml`,
`examples/polly/skills/cross-review/SKILL.md`, and
`examples/polly/skills/fanout/SKILL.md`. Added file:
`examples/polly/skills/auto-merge/SKILL.md`.

Omnigent is Copyright (2026) Databricks, Inc., licensed under the Apache
License 2.0. See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).
