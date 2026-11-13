# CI/CD and repository rules

What runs on GitHub for this repository and which rules protect `main` and release tags. The [root README](../../README.md#repository-settings) lists the settings and rulesets that are created by hand first and describes the git hooks on your machine.

## On a pull request

A pull request into `main` starts these checks. With the optional `dev` branch a pull request into `dev` starts them too.

| Check | Workflow | Required to merge |
| --- | --- | --- |
| `verify / verify` | `verify.yml`, passes only when `verify / uv` and `verify / base / verify` passed | yes |
| `verify / uv` | `verify.yml`, runs ruff, mypy, pytest and `uv build` | not on its own, `verify / verify` fails when it fails |
| `verify / base / verify` | `verify.yml`, lints Dockerfiles with hadolint, shell scripts with shellcheck and workflow files with actionlint | not on its own, `verify / verify` fails when it fails |
| `review / review` | `scan-dependencies.yml`, fails on new dependencies with a vulnerability of severity high or critical | yes |
| `analyze / analyze` | `scan-codeql.yml`, CodeQL scan of the Python code | not as a check, but the ruleset on `main` blocks the merge on CodeQL errors or security alerts of severity high or above |
| `CodeQL` | Posted by GitHub code scanning under `Code scanning results` after `scan-codeql.yml` uploaded its results. Lists the alerts found in the pull request | no |

## Workflows

The workflow files only hold the triggers, their jobs live as reusable workflows in [aktin-github-scripts](https://github.com/aktin/aktin-github-scripts/tree/main/.github/workflows) and are called at `@main`.

What each workflow does and when it runs. Schedules are in UTC and run on `main`:

| Workflow | Pull request | Push to `main` | Schedule | Other | What happens |
| --- | --- | --- | --- | --- | --- |
| `verify.yml` | yes | yes | | | Starts the shared `verify-python.yml`. Its job `uv` installs the dependencies from `uv.lock` and runs `ruff format --check`, `ruff check`, `mypy`, `pytest` and `uv build`. Its job `base` starts the shared `verify-common.yml`, which checks Dockerfiles with hadolint, shell scripts with shellcheck and the workflow files with actionlint. It fails when hadolint reports an error, shellcheck a warning or worse, or actionlint any finding. Its job `verify` fails when one of the two did not pass. A new run for the same branch or pull request cancels the one that is still running |
| `scan-dependencies.yml` | yes | | | | Compares the dependencies before and after the pull request and fails when a newly added one has a known vulnerability of severity high or critical. When it fails it writes the findings as a comment into the pull request |
| `scan-codeql.yml` | yes | yes | Mondays 03:00 | | Scans the Python code with CodeQL and reports the findings in the Security tab. A new run for the same branch or pull request cancels the one that is still running |
| `scan-scorecard.yml` | | yes | Mondays 04:00 | | Rates the security practices of the repository, for example branch rules, pinned actions, token permissions and code review, publishes the score to OpenSSF and feeds the badge in the root README. The findings also appear in the Security tab. It changes nothing |
| `release.yml` | | | | tag `v*` | Runs three jobs one after the other. `verify` runs the same checks as `verify.yml`. `build` starts the shared `build-python.yml`, which builds wheel and sdist of the tagged commit with the version taken from the tag (tag `v1.2.3` gives version `1.2.3`, `pyproject.toml` stays at `0.0.0`), writes the SBOM `sbom-python.cdx.json` with the runtime dependencies from `uv.lock` and attests build provenance and the SBOM at GitHub. `publish` starts the shared `publish-release.yml`, which writes the release notes from the commit history with git-cliff and creates the GitHub Release with wheel, sdist and SBOM attached. It fails when it finds no file to attach |

Dependabot is configured in `.github/dependabot.yml`. Once a week it opens one grouped pull request per ecosystem, one for the Python dependencies (`uv`) and one for the actions referenced in the workflow files (`github-actions`). They go against `main`, the default branch, as long as `dependabot.yml` sets no `target-branch`. The shared workflows pin their actions to a commit SHA. Those pins are updated in aktin-github-scripts, not here. Dependabot alerts and security updates are switched on in the repository settings and report known vulnerabilities in dependencies that are already in use. Security updates always open their pull requests against `main`, also when the optional `dev` branch is used.

## Gates on `main`

Branch ruleset `default` on `main`:

| Rule | Meaning |
| --- | --- |
| Pull request required | No direct pushes. Squash merge only, 1 approval, code owner review, approval after the last push, all review threads resolved |
| Required checks | `verify / verify` and `review / review` must pass |
| Code scanning | CodeQL must have analysed the head commit without errors or security alerts of severity high or above |
| Signed commits | Every commit on `main` is signed. The squash commit is signed by GitHub. Unsigned commits in a pull request can block the merge |
| No force push, no delete | History on `main` is append-only |

With the optional `dev` branch a branch ruleset `dev` applies to `dev`: pull request required with squash merge only, 1 approval and resolved review threads, the same required checks, signed commits, no force push and no delete. `main` then takes merge commits only.

Approvals come from people. GitHub Actions cannot create or approve pull requests, and no app or bot is on the bypass list. Organisation admins can bypass every rule. In a repository under a personal account the repository admin can. Other branches have no rules.

## Gates on tags `v*`

Tag ruleset `release-tags`: only maintainers and admins can create, move or delete release tags. A tag starts the release workflow, so this decides who can cut a release.
