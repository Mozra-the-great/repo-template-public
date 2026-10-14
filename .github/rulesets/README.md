# Branch rulesets

`default.json` is the ruleset shape used by nagellacke and homelab-infra,
generalised for this scaffold. It lives here so that the merge gate is
traceable in the repo and not only in the web UI.

**It is not applied automatically and must not be applied by a rollout PR.**
No workflow restores it; applying it is a repository *setting* and is done by the
owner, after confirming the exact check names (convention C10). If the UI and
this file drift apart, re-export after every change:

```bash
gh api repos/<owner>/<repo>/rulesets/<id> \
  --jq '{name, target, enforcement, conditions, bypass_actors, rules}' \
  | python3 -c 'import json,sys; print(json.dumps(json.load(sys.stdin), indent=2, ensure_ascii=False, sort_keys=True))' \
  > .github/rulesets/default.json
```

Applying it for the first time (owner, once the workflows ran green on `main`):

```bash
gh api --method POST repos/<owner>/<repo>/rulesets --input .github/rulesets/default.json
```

## What the ruleset enforces

| Rule | Effect |
|---|---|
| `pull_request` | No direct push to the default branch, everything through PRs; **every review thread must be resolved** |
| `required_status_checks` | `build`, `workflow-lint` and `gitleaks` must pass, each bound to `integration_id: 15368` (GitHub Actions) so no other app can report a same-named check |
| `non_fast_forward` | No force-push |
| `deletion` | The default branch cannot be deleted |

`required_approving_review_count` is **0**: GitHub forbids approving your own PR
and a solo-maintained repo has exactly one maintainer, so a required approval
would not be strict, it would be unsatisfiable. The thread-resolution rule is
the real review gate instead.

`bypass_actors` is **empty** on purpose. An entry for the admin role would let the
owner (and any agent working with the owner's token) past the rules, which makes
the gate useless for exactly the cases it was built for. If you really have to
get around it in an emergency, set the ruleset to `disabled` in the UI for a
moment: a visible, deliberate step instead of a silent exception.

## Check names

The check context is the **job id** of the workflow (none of these jobs sets a
`name:`, so id and context are identical):

| Context | Workflow | Notes |
|---|---|---|
| `build` | `ci.yml` | stack lint/test; runs on every PR (no `paths:` filter, convention C1) |
| `workflow-lint` | `workflow-lint.yml` | actionlint + zizmor |
| `gitleaks` | `secret-scan.yml` | PR-range secret scan |
| `claude-review` | `claude-code-review.yml` | **optional**, not in `default.json`, see below |

Confirm the names against a real run before applying (`gh pr checks <n>` on a
PR, or `gh api repos/<owner>/<repo>/commits/<sha>/check-runs --jq '.check_runs[].name'`).

### Adding `claude-review` as a required check

Only in repos that set the `CLAUDE_CODE_OAUTH_TOKEN` secret and receive real PRs
(decision D3). Add this entry to `required_status_checks`:

```json
{ "context": "claude-review", "integration_id": 15368 }
```

Things to know before you do:

- **Green does not always mean reviewed.** Without the secret, and on fork PRs,
  the job skips its steps and ends green. As a required check it then only
  means "skipped". Dependabot PRs skip the whole job (`if:`), and GitHub counts a
  skipped job as passing.
- `synchronize` has to stay in the workflow trigger while it is required. GitHub
  evaluates required checks against the head commit, and a job that does not run
  on a pushed commit never reports there (nagellacke #312 was blocked forever by
  exactly that).
- The "Fail on findings" step turns the check red when Claude posted a finding.
  For a false positive, resolve the thread without a code change and re-run the
  job.
- If you later delete `claude-code-review.yml`, remove the entry too (C4).

## Auto-merge

Native auto-merge (`gh pr merge --auto --squash`) needs two things the PR cannot
set: the repo setting **Allow auto-merge** and at least one required status
check. Both are owner decisions (D7). Without them `--auto` fails with an error,
and the PR is merged by hand once the checks are green.

## Cost

The ruleset itself is free. The checks it requires run as Actions jobs, which
are free on a public repo (on a private one they draw on the account's Actions
minutes, see `docs/TEMPLATE-SETUP.md`).
