# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- CI building blocks from nagellacke and homelab-infra, stack-agnostic: `ci.yml`
  skeleton (job `build`), `workflow-lint.yml` (actionlint + zizmor),
  `secret-scan.yml` (gitleaks) with `.gitleaks.toml`, `dependabot.yml`, opt-in
  labels-as-code (`labels.json`, `labels.yml`, `scripts/ci/`) and a ruleset
  template under `.github/rulesets/` (not auto-applied).
- `docs/TEMPLATE-SETUP.md`: secrets, runner choice, rulesets and costs of a new repo.

### Fixed
- `claude.yml`: the job only starts for `@claude` mentions from an `OWNER`,
  `MEMBER` or `COLLABORATOR` (job-level `author_association` gate), so drive-by
  mentions on a public repo cannot start a run. The action's own write-permission
  check remains as the second layer.
- `claude.yml`: explicit `claude_args` tool allowlist
  (`--allowedTools Skill,Bash,Read,Edit,Write,Grep,Glob`) instead of the action's
  unrestricted default; documented in `docs/TEMPLATE-SETUP.md`.

### Changed
- `claude-code-review.yml` replaced by the hardened nagellacke version
  (`--comment`, inline-comment allowlist, `--disallowedTools`, concurrency group,
  execution-log artifact, fail-on-findings). Both Claude workflows now skip
  green when `CLAUDE_CODE_OAUTH_TOKEN` is not set instead of failing red.
- All actions pinned by commit SHA.
