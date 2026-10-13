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

[![CI](https://github.com/{{REPO_SLUG}}/actions/workflows/ci.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/ci.yml)
[![CodeQL](https://github.com/{{REPO_SLUG}}/actions/workflows/codeql.yml/badge.svg)](https://github.com/{{REPO_SLUG}}/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/{{REPO_SLUG}}/badge)](https://securityscorecards.dev/viewer/?uri=github.com/{{REPO_SLUG}})
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/{{REPO_SLUG}})](https://github.com/{{REPO_SLUG}}/releases)

# {{DESCRIPTION}}

## Stack

uv-managed Python package (src layout), ruff for linting and formatting, mypy strict, pytest with coverage gate, lefthook with shared hooks from aktin-github-scripts, tag-driven releases via git-cliff, and a hardened GitHub Actions pipeline (CI, CodeQL, Scorecard, dependency review, SBOM, build attestations).

## Getting started

Prerequisites: [uv](https://docs.astral.sh/uv/) on PATH (it installs the interpreter pinned in `.python-version`). The shared git hooks are Python scripts and need `python3` on PATH.

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

Allowed types are `feat fix docs style refactor perf test chore`, the subject starts lowercase and has no trailing period, the header stays at most 100 characters. Cut a release by pushing a `vX.Y.Z` tag.

## Repository settings

Generating a repo from a template copies files and branches only, never settings, so these one-time steps stay manual. Do them after `scripts/init.py`.

| Setting | Where and what to do | Why |
| --- | --- | --- |
| Branch ruleset on `main` | Settings → Rules → Rulesets. New branch ruleset targeting `main`: require a pull request, require the `CI` check, block force pushes | Keeps unreviewed or broken code off `main` |
| Actions bypass on the ruleset | Same screen, add GitHub Actions to the bypass list | The release workflow pushes a `chore(release)` commit back to `main`. Without the bypass that push is rejected and the release stops |
| Tag ruleset on `v*` | Same screen, new tag ruleset targeting `v*`, restrict creation to maintainers | Releases are tag-driven, without the rule anyone with push access can cut one |
| Workflow permissions | Settings → Actions → General → Read and write | The release workflow creates the GitHub Release and pushes the changelog commit |
| Fork PR approval | Settings → Actions → General, require approval for outside collaborators | Keeps untrusted fork code from running workflows before a maintainer has looked |
| Merge settings | Settings → General → Pull Requests. Allow squash merging only, default the commit message to the PR title, auto-delete head branches | One conventional subject per PR lands on `main`, which is what git-cliff reads for the changelog |
| Code scanning | Runs from `codeql.yml`, keep "Default setup" off in Settings → Code security | Default setup conflicts with the advanced workflow, private repos need GitHub Advanced Security for the alerts to appear |
| Secret scanning + push protection | Settings → Code security, on by default for public repos | Blocks committed credentials |
| Dependabot alerts | Settings → Code security, enable Dependabot alerts and security updates | `dependabot.yml` only schedules version updates, the vulnerability alerts and auto-fix PRs are separate toggles |
| Private vulnerability reporting | Settings → Code security, enable | Gives researchers a private channel instead of a public issue |
| Template toggle | Leave "Template repository" off if this repo was made from the template | Prevents re-templating |
| Housekeeping | Settings → General. Set description and topics, untick Wikis and Projects if unused | Discoverability, fewer places for stale content |

## How the infra works

| Component | What it does | Why |
| --- | --- | --- |
| uv | Package and environment manager, single lockfile, interpreter pinned in `.python-version` | Same environment locally and in CI |
| ruff | Owns linting and formatting | One tool, no rule conflicts |
| mypy strict | Type checks `src` and `tests` | Keeps the package honest from day one |
| pytest + coverage | Unit tests with an 80% line gate (`fail_under`) | Regressions surface locally and in CI |
| lefthook + shared hooks | pre-commit formats and lints staged files, runs gitleaks when installed and the shared hooks from aktin-github-scripts (data files, large files, commit size), commit-msg enforces Conventional Commits via the shared hook, pre-push runs typecheck, test, build | Catch problems locally, keep history parseable for the changelog |
| CI | `ci.yml` calls the reusable `verify.yml` on push and PR, cancels superseded runs, least-privilege token | Fast, safe feedback |
| Security scans | CodeQL, dependency-review, OpenSSF Scorecard, secret scanning, all surfaced in the Security tab | Catch code and supply-chain issues early |
| Releases | Tag `vX.Y.Z` triggers git-cliff changelog and notes, version bump committed back, CycloneDX SBOM, SLSA attestations, sdist + wheel attached to the Release | Reproducible, auditable releases from commit history |
| Dependabot | Weekly grouped uv and github-actions updates | Stay patched, grouping avoids partial bumps that break paired steps such as CodeQL |
| SHA-pinned actions | Every action is pinned to a full commit SHA with a version comment | A moved tag cannot swap the code, Dependabot keeps the SHAs current |
| Pinned Python | Interpreter version fixed in `.python-version`, floor in `requires-python` | Same runtime locally and in CI |

## Project structure

```
src/python_starter/ the package (renamed by init.py)
tests/              pytest suite
.github/workflows/  CI, CodeQL, Scorecard, dependency-review, release
SECURITY.md          how to report a vulnerability
scripts/init.py     one-time template initializer (removed after first run)
```
