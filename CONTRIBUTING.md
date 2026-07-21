# Contributing

## Prerequisites

[uv](https://docs.astral.sh/uv/) is the package and environment manager. The
interpreter version is pinned in `.python-version`.

```sh
uv sync
uv run lefthook install
```

This installs dependencies, creates the venv, and sets up the git hooks.

## Development

```sh
uv run pytest        # unit tests with coverage
uv run mypy          # type check
uv run ruff check .  # lint
uv run ruff format . # format
uv build             # sdist + wheel
```

## Commits

Messages follow [Conventional Commits](https://www.conventionalcommits.org),
enforced by commitizen through the `commit-msg` hook.

## Pull requests

Make sure `uv run ruff check .`, `uv run mypy`, and `uv run pytest` pass. The
pre-push hook runs those plus the build.

## Releases

Push a `vX.Y.Z` tag. The release workflow builds sdist and wheel, generates
the changelog and notes, produces an SBOM and attestations, and publishes the
GitHub Release.
