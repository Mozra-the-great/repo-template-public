# Contributing

This is primarily a personal/solo-maintained project. If you're proposing a
change:

## Workflow

1. Never commit directly to `main` — create a feature branch.
2. Branch names: English, kebab-case, prefixed by type — `feat/...`,
   `fix/...`, `chore/...`, `docs/...`, `refactor/...`, `test/...`.
3. Commits follow [Conventional Commits](https://www.conventionalcommits.org/):
   `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`.
4. Open a pull request against `main`. CI (and, where configured, Claude Code
   review) must pass before merge.
5. No `git push --force` on `main`.

## CI

Every PR runs the checks below (none has a `paths:` filter, so they run on
docs-only PRs too):

| Check | What it does |
|---|---|
| `build` | Stack lint/test from `.github/workflows/ci.yml` (plus the label-script tests while `scripts/ci/` exists) |
| `workflow-lint` | actionlint + zizmor over the workflows and `dependabot.yml` |
| `gitleaks` | Secret scan over the PR range, using `.gitleaks.toml` |
| `claude-review` | Optional Claude Code review; needs the `CLAUDE_CODE_OAUTH_TOKEN` secret and is **skipped (green)** without it |

Workflow rules: actions are pinned by full commit SHA with a `# vX.Y.Z` comment,
event data (`github.event.*`, branch names, titles) goes through `env:` and
never into `run:` via `${{ }}`, and `permissions:` stays minimal. Run
`actionlint` and `zizmor` locally before pushing a workflow change; the setup
steps, costs and the open runner/ruleset decisions are in
[`docs/TEMPLATE-SETUP.md`](docs/TEMPLATE-SETUP.md).

## Reporting bugs / requesting features

Use the issue templates under `.github/ISSUE_TEMPLATE/`.

## Security issues

Do not open a public issue — see [`SECURITY.md`](SECURITY.md).
