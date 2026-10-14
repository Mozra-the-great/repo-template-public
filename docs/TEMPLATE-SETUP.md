# Setting up a repo from this template

This file describes what a repository created from the template has to configure
before its CI is useful. **Delete it from the new repo once you are done**, or
keep it as a record. Nothing here is applied automatically: secrets, rulesets
and repository settings are owner decisions and never part of a workflow PR.

The building blocks (B1-B9), conventions (C1-C11) and open decisions (D1-D7)
referenced below are defined in the private umbrella issue
`Mozra-the-great/repo-template#2`.

## What the template ships

| Block | Files | Needs | Cost |
|---|---|---|---|
| B1 Claude review | `.github/workflows/claude-code-review.yml` | secret `CLAUDE_CODE_OAUTH_TOKEN` (optional) | A full review per push to a PR draws on the Claude 5h/7d budget of the account behind the token; Actions minutes in a private repo |
| B2 `@claude` mention | `.github/workflows/claude.yml` | same secret (optional) | Runs only on a mention, cheap |
| B3 CodeQL | none - GitHub *default setup*, see [Scanning](#code-and-secret-scanning) | a repo setting | Public: free. Private: unavailable |
| B4 Labels as code | `.github/labels.json`, `.github/workflows/labels.yml`, `scripts/ci/labels.py`, `scripts/ci/test_labels.py` | repo variable `LABELS_ENABLED=true` (opt-in) | Many short jobs (every issue/PR event plus a daily cron); only worth it where issues are used |
| B5 Dependabot | `.github/dependabot.yml` | nothing | Dependabot runs as Actions jobs: free on public, minutes on private |
| B6 Secret scan | `.github/workflows/secret-scan.yml`, `.gitleaks.toml` | nothing | One short job per PR |
| B7 Lint / test | `.github/workflows/ci.yml` (job `build`), `.github/workflows/workflow-lint.yml` (job `workflow-lint`) | uncomment your stack block | 1-3 minutes per run |
| B8 Merge gate | `.github/rulesets/default.json` + README | owner applies it | none |
| B9 Hygiene | issue/PR templates, `CONTRIBUTING.md`, `SECURITY.md`, `CLAUDE.md`, `.editorconfig`, `.gitattributes` | fill in the placeholders | none |

The only secret the scaffold knows is `CLAUDE_CODE_OAUTH_TOKEN`. `GITHUB_TOKEN`
is provided by Actions itself.

## Steps

1. **Create the repo** with *Use this template*. GitHub copies files, not
   settings: secrets, variables, rulesets and security toggles start empty.
2. **Fill in the placeholders:** `CLAUDE.md` (`<PROJECT_NAME>`, Type, Stack,
   Structure, Commands, Notes), `README.md`, `NOTICE`.
3. **Enable your stack in `ci.yml`:** uncomment one block (Python, Node, Shell)
   or add your own, and delete the others. Keep the job id `build`: it is the
   stable name of the required check. Do **not** add a workflow-level `paths:`
   filter (C1, see below).
4. **Dependabot:** in `.github/dependabot.yml` uncomment the ecosystems the repo
   uses and keep `cooldown` on each of them. Merge Dependabot PRs only after
   `build` ran on them (C9).
5. **Labels (optional):** edit `.github/labels.json` (add your `area/` labels and
   the matching `paths` / `title_scopes`), then set the repository variable
   `LABELS_ENABLED` to `true` and run the *Labels* workflow once with
   *Run workflow* so the labels get created. Once your rules cover every
   tracked file, set `require_path_coverage` to `true` in `labels.json` so a new
   uncovered path fails the tests. Not wanted: delete the labels files listed
   in the table above.
6. **Claude review (optional, D3):** only for repos that get real PRs. The owner
   sets the secret in the repo settings, without putting the value on a command
   line or in a log:

   ```sh
   gh secret set CLAUDE_CODE_OAUTH_TOKEN --repo <owner>/<repo>
   ```

   (`gh` prompts for the value.) Without the secret both Claude workflows skip
   their steps and end **green**, so a new repo has no red check for a missing
   secret. A green `claude-review` without the secret means "skipped", not
   "reviewed".

   `claude.yml` additionally only starts for mentions from an `OWNER`, `MEMBER` or
   `COLLABORATOR` (job-level `author_association` gate), so drive-by mentions on a
   public repo never reach the action; keep that `if:` when you edit the file.
   The file also passes an explicit tool allowlist to the action
   (`claude_args: '--allowedTools Skill,Bash,Read,Edit,Write,Grep,Glob'`) instead
   of the unrestricted default; widen it deliberately if your project needs more
   tools rather than removing it.
7. **Pick a runner** (see next section) before you make any check required.
8. **Rulesets (D7):** follow `.github/rulesets/README.md` and the order below.
   A rollout PR never changes repository settings.

### Order for a new repo (C10)

1. Workflow PR(s) merged only when green.
2. Verify the job names on `main` (`build`, `workflow-lint`, `gitleaks`, and
   `claude-review` if used).
3. The owner confirms the **exact** settings: secrets, ruleset with the named
   checks, CodeQL default setup, `allow_auto_merge`, `delete_branch_on_merge`.
4. Apply them. Only then do `gh pr merge --auto --squash` and the ruleset
   interact (auto-merge needs the setting *and* at least one required check).

## Runner choice

**This is a public repo.** Hosted `ubuntu-latest` runners are free for public
repositories, so every workflow works as shipped and no runner decision is
needed. Never register a self-hosted runner for a public repo (C7): a fork PR
would run its code on your machine.

If you create a **private** repo from this template, the situation is different
(decisions D1/D2): hosted Actions minutes were exhausted on 2026-09-24 and GitHub
then started no hosted job at all, and whether they are available again is not
settled (the billing API needs the `user` scope). Observation, not an answer: on
2026-10-14 the CI of the private sibling's own PR did start on hosted runners and
passed, so minutes were available at that moment; that can change with the
billing cycle or the spending limit. If hosted minutes are unavailable:

- Every workflow says `runs-on: ubuntu-latest` and will not start without hosted
  minutes.
- Do **not** make `build`, `workflow-lint` or `gitleaks` required checks until a
  test PR proves they start (C4): a required check that cannot start blocks
  every PR.
- D1 options: (a) register a self-hosted runner per repository and change
  `runs-on` in every workflow (private repos only, no secrets, no LAN
  credentials; declare the label to actionlint in `.github/actionlint.yaml`, and
  the Claude workflows also need the Claude Code CLI installed on that runner);
  (b) markdown-only repos use a local pre-commit hook only; (c) a spending
  limit; (d) make the repo public.
- Validate locally (see [Local validation](#local-validation)).

## Code and secret scanning

This is a public repo, so GitHub's own scanning is **free**. These are repository
settings, so the owner enables them (the template cannot, and a PR never does):

- **Secret scanning and push protection:** Settings -> Code security. Push
  protection blocks a push that contains a known token before it lands; the
  `gitleaks` job in this template is a second layer over the PR range.
- **CodeQL:** Settings -> Code security -> Code scanning -> Set up -> *Default*.
  Default setup needs no workflow file and detects the languages itself; use an
  *advanced* setup (a `codeql.yml` workflow) only when you need custom queries
  or a custom build. The first run shows the real check names (for example
  `Analyze (python)`); add them to the ruleset only after you have seen them.

If you create a **private** repo from this template, these are not available
(they need GitHub Advanced Security; the settings report `null`) and `gitleaks`
is the only scanner.

gitleaks as a local pre-commit hook (same version as CI, same `.gitleaks.toml`):

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: 83d9cd684c87d95d656c1458ef04895a7f1cbd8e  # frozen: v8.30.1
    hooks:
      - id: gitleaks
```

## Conventions the workflows follow

Keep them when you add workflows:

- **C1** Workflows that provide required checks have no workflow-level `paths:`
  filter. A required check that never runs stays pending forever. Skip with a
  job-level `if` instead (a skipped job counts as passing).
- **C2** Keep `synchronize` in the `claude-review` trigger while it is required.
- **C5** Actions are pinned by full commit SHA with a `# vX.Y.Z` comment;
  Dependabot updates them weekly with a 7-day cooldown.
- **C6** Event data (`github.event.*`, branch names, PR titles) goes through
  `env:`, never `${{ }}` inside `run:`. `pull_request_target` only for the
  labels script, and only checking out the base branch. Default
  `permissions: contents: read` (or `{}`), and `persist-credentials: false`
  wherever the job does not push. No `secrets.*` in a job-level `if:` (invalid):
  read it into an env var in a step and gate on the result.
- **C8** Agents never read, print or commit secret values.

## Local validation

`workflow-lint` runs actionlint and zizmor in CI. The same versions run locally:

```sh
python -m venv .venv-lint
.venv-lint/bin/pip install zizmor==1.30.1 "actionlint-py==1.7.12.*" shellcheck-py
.venv-lint/bin/actionlint
GH_TOKEN="$(gh auth token)" .venv-lint/bin/zizmor .   # token via env only, never as an argument
python3 -m unittest discover -s scripts/ci -p "test_*.py"
```

(On Windows the binaries are in `.venv-lint\Scripts\`.) `.venv-lint/` is
git-ignored.

## Open decisions this template does not make

D1 private-repo runner strategy, D2 hosted-minutes status, D3 which repos get
the Claude review, D6 copy vs. central reusable workflows, D7 rulesets and
`delete_branch_on_merge`. The template ships working defaults for a public
repo and states the caveats for a private one; it decides none of them.
