# {{PROJECT_NAME}}

<!-- template:start -->

> **This repo was generated from the `python-starter` template.** Run the one-time setup before anything else:
>
> ```sh
> uv sync                        # installs deps and creates the venv
> uv run python scripts/init.py  # fills in name, slug, author, owner; strips this banner; self-deletes
> uv run lefthook install        # installs the git hooks
> ```
>
> `init.py` reads your `git remote` to guess the slug, prompts for the rest, rewrites the placeholders (`{{PROJECT_NAME}}`, `{{REPO_SLUG}}`, `{{DESCRIPTION}}`, `{{AUTHOR}}`, `{{OWNER}}`) across the repo, and renames the `python_starter` package. After it runs, run `uv sync`, review the diff, commit, then do the one-time GitHub steps under [Repository settings](#repository-settings).

<!-- template:end -->

[![Verify](https://github.com/{{REPO_SLUG}}/actions/workflows/verify.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/verify.yml)
[![CodeQL](https://github.com/{{REPO_SLUG}}/actions/workflows/scan-codeql.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/scan-codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/{{REPO_SLUG}}/badge)](https://securityscorecards.dev/viewer/?uri=github.com/{{REPO_SLUG}})
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/{{REPO_SLUG}})](https://github.com/{{REPO_SLUG}}/releases)

# {{DESCRIPTION}}

## Stack

uv-managed Python package (src layout), ruff for linting and formatting, mypy strict, pytest with coverage gate, lefthook with shared hooks from aktin-github-scripts, tag-driven releases via git-cliff, and a hardened GitHub Actions pipeline (CI, CodeQL, Scorecard, dependency review, SBOM, build attestations).

## Getting started

Prerequisites: [uv](https://docs.astral.sh/uv/) on PATH (it installs the interpreter pinned in `.python-version`). The shared git hooks need `python3`, `gitleaks`, `actionlint`, `hadolint` and `shellcheck` on PATH. A missing tool fails the commit. actionlint, hadolint and shellcheck are only called when a workflow file, Dockerfile or shell script is staged.

```sh
uv sync                  # deps + venv
uv run lefthook install  # git hooks (one-time)
uv run pytest            # unit tests with coverage
uv run mypy              # type check
uv run ruff check .      # lint
uv run ruff format .     # format
uv build                 # sdist + wheel
```

The coverage report lands in `htmlcov/index.html`.

Commits must follow [Conventional Commits](https://www.conventionalcommits.org), enforced by the commit-msg hook:

```
<type>(<optional scope>): <subject>
```

Allowed types are `feat fix docs style refactor perf test chore`, the subject starts lowercase and has no trailing period, every line of the message stays at most 100 characters.

Cut a release by pushing a `vX.Y.Z` tag on `main`, for example version 1.2.3:

```sh
git switch main
git pull
git tag v1.2.3
git push origin v1.2.3
```

## Repository settings

Generating a repo from a template copies files and branches only, never settings, so these one-time steps stay manual. Do them after the commit from `scripts/init.py` is pushed to `main`. Once the ruleset on `main` is active it rejects direct pushes.

| Setting | Where and what to do | Why |
| --- | --- | --- |
| Branch ruleset on `main` | Settings → Rules → Rulesets. New branch ruleset `default` targeting the default branch: restrict deletions, block force pushes, require signed commits, require a pull request (1 approval, code owner review, dismiss stale approvals on push, approval of the most recent push, all conversations resolved, squash only), require the status checks `verify / verify` and `review / review`, require CodeQL code scanning results (security alerts high or higher, alerts errors). Set the enforcement status to Active and add `Organization admin` to the bypass list | Keeps unreviewed, unsigned or broken code off `main` |
| Tag ruleset on `v*` | Same screen, new tag ruleset `release-tags` targeting the tag pattern `v*`: restrict creations, updates and deletions, enforcement status Active, bypass for admins and maintainers only | Releases are tag-driven, without the rule anyone with push access can cut one |
| Integration branch `dev` (optional) | Only if features should collect on `dev` before a release lands on `main`. Create `dev` from `main`. New branch ruleset `dev` targeting the branch `dev`: restrict deletions, block force pushes, require signed commits, require a pull request (1 approval, all conversations resolved, squash only), require the status checks `verify / verify` and `review / review`, enforcement status Active, `Organization admin` on the bypass list. In the ruleset `default` target the branch `main` instead of the default branch and allow merge commits only. In Settings → General → Pull Requests also allow merge commits. In `dependabot.yml` add `target-branch: dev` to every entry. The workflows already run their checks on pull requests into `dev`. | Feature PRs squash into `dev`, `dev` merges into `main` with a merge commit so the two never diverge, `main` keeps mirroring the released state |
| Bypass list under a personal account | In every ruleset add `Repository admin` to the bypass list instead of `Organization admin` | A personal account has no organisation admin. Without the bypass the only maintainer cannot merge the own pull requests, because 1 approval and a code owner review are required |
| Workflow permissions | Settings → Actions → General → Read repository contents and packages permissions, do not allow GitHub Actions to create or approve pull requests | Every workflow declares its own `permissions`, the release workflow requests `contents: write` itself |
| Fork PR approval | Settings → Actions → General, require approval for all external contributors | Keeps untrusted fork code from running workflows before a maintainer has looked |
| Pull request creation | Settings → General → Features → Pull requests, select Collaborators only | Only people with access to the repository can open pull requests |
| Merge settings | Settings → General → Pull Requests. Allow squash merging only, default to the pull request title with a blank message, always suggest updating pull request branches, auto-delete head branches | One conventional subject per PR lands on `main`, which is what git-cliff reads for the release notes |
| Commit sign-off | Settings → General, require contributors to sign off on web-based commits | Commits made in the browser carry the same attribution as local ones |
| Immutable releases | Settings → General → Releases, enable release immutability | The files and the tag of a published release cannot be changed afterwards |
| Code scanning | Runs from `scan-codeql.yml`, keep "Default setup" off in Settings → Advanced Security | Default setup conflicts with the advanced workflow |
| Private repository | CodeQL and the dependency review need GitHub Code Security. Artifact attestations need GitHub Enterprise Cloud, without it set `attest: false` on the job `build` in `.github/workflows/release.yml` as in the example below the table | Without Code Security `review / review` and the code scanning rule cannot pass. Without Enterprise Cloud the release fails. `attest: false` skips the SBOM and the attestations |
| Secret scanning + push protection | Settings → Advanced Security, on by default for public repos | Blocks committed credentials |
| Dependabot alerts | Settings → Advanced Security, enable Dependabot alerts and security updates | `dependabot.yml` only schedules version updates, the vulnerability alerts and auto-fix PRs are separate toggles |
| Labels | Issues → Labels. Delete `duplicate` and `invalid`, add `critical` (`B60205`, "Blocking release or core functionality") and `minor` (`FEF2C0`, "Low impact issue") | Same triage labels in every repo |
| Template toggle | Leave "Template repository" off if this repo was made from the template | Prevents re-templating |
| Housekeeping | Settings → General. Set description and topics, untick Wikis and Projects if unused | Discoverability, fewer places for stale content |

The job `build` in `.github/workflows/release.yml` of a private repository without GitHub Enterprise Cloud:

```yaml
  build:
    needs: verify
    permissions:
      contents: read
      id-token: write
      attestations: write
    uses: aktin/aktin-github-scripts/.github/workflows/build-python.yml@main
    with:
      attest: false
```

## How the infra works

| Component | What it does |
| --- | --- |
| uv | Package and environment manager, single lockfile, interpreter pinned in `.python-version` |
| ruff | Owns linting and formatting |
| mypy strict | Type checks `src` and `tests` |
| pytest + coverage | Unit tests with an 80% line gate (`fail_under`) |
| lefthook | `lefthook.yml` defines the format, lint and pre-push hooks and loads all other hooks from aktin-github-scripts. pre-commit: ruff formats and lints the staged `*.py` files and stages the fixes, gitleaks scans for secrets, actionlint, hadolint and shellcheck lint the staged workflow files, Dockerfiles and shell scripts, the shared scripts reject data files outside `tests/fixtures/`, files larger than 500 KB and commits with more than 200 changed lines. commit-msg: checks the Conventional Commits format. pre-push: `uv run mypy`, `uv run pytest` and `uv build` |
| Pinned Python | Interpreter version fixed in `.python-version`, floor in `requires-python` |
| Workflows, Dependabot and rulesets | Checks on pull requests, security scans, the release on a tag and the rules on `main` and release tags are described in [.github/workflows/README.md](.github/workflows/README.md) |

## Project structure

```
src/python_starter/ the package (renamed by init.py)
tests/              pytest suite
.github/workflows/  workflow files and the README that describes them
SECURITY.md          security policy
scripts/init.py     one-time template initializer (removed after first run)
```
