# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **composite GitHub Action** (not a standalone application) that provides the Mission Critical Vulnerability Scanner (MCVS) for Python projects. The action performs security scanning, linting, testing, and optional binary building for Python codebases.

## Architecture

### Composite Action Structure

The action is defined in `action.yml` and executes as a series of composite steps:

1. **Python Environment Setup**: Installs Python version from `.python-version`
2. **Security Scanning**: Uses Anchore scan-action (Grype) to detect vulnerabilities
3. **Resolve**: Picks the package manager (uv when `uv.lock` exists, else pip) and validates inputs
4. **Dependency Installation**: `uv sync --all-packages --frozen`, or `requirements.txt` if present
5. **Testing**: Plain pytest for uv projects or when `test-paths`/`pytest-args` is set; otherwise the legacy `test.py` run
6. **Code Linting**: ruff (default for uv) or Flake8 with an error threshold (default for pip)
7. **Type Checking**: Optional mypy or pyright
8. **Binary Building**: Conditionally builds PyInstaller binaries on tag releases (pip projects only)

Project tools (ruff, mypy, pyright, pytest) run from the project environment via the
`run` output of the resolve step (`uv run --no-sync` or `python3 -m`), so their versions
and config come from the project. User inputs reach the shell through `env:`, never
through `${{ }}` inside `run:`.

### Key Design Decisions

- **Composite vs Docker**: Uses `using: composite` to avoid Docker overhead and enable caching
- **Conditional Execution**: Steps like testing and binary building only run when applicable
- **Token Authentication**: Requires GitHub token to attach binaries to releases
- **Version Pinning**: All tools are pinned to specific versions for reproducibility

## Version Constraints

**CRITICAL**: Actions are pinned by commit SHA in `action.yml`; pip packages are pinned with hashes in `configs/pip/*/requirements.txt`:

- `actions/setup-python@v7.0.0` (action.yml:22)
- `anchore/scan-action@v7.4.2` (action.yml:29)
- `flake8==7.4.1` (configs/pip/flake8/requirements.txt)
- `pyinstaller==6.22.3` (configs/pip/pyinstaller/requirements.txt)
- `svenstaro/upload-release-action@2.11.5` (action.yml:104)

When updating dependencies:
- Actions: update the commit SHA and the `# vX` comment in `action.yml`
- Pip packages: update the version and all `--hash` entries (including transitive deps, required by `--require-hashes`)
- Dependabot automatically creates PRs for GitHub Actions updates (see `.github/dependabot.yml`)
- Python package versions must be updated manually

## Testing Changes

This action is tested via PR validation:

```yaml
# Validation happens automatically on PRs via .github/workflows/mcvs-pr-validation.yml
# Uses schubergphilis/mcvs-pr-validation-action@v0.2.2
```

To test locally before committing:

```bash
# Test YAML linting (matches action behavior)
pip install yamllint==1.37.1
yamllint .

# Validate action.yml structure
# No local validation tool - rely on PR validation workflow
```

## Dependency Management

### Dependabot Configuration

Dependabot is configured for GitHub Actions only (`.github/dependabot.yml`):
- Runs weekly checks
- 5-day cooldown between updates
- Groups all GitHub Actions updates together

**Note**: Python package dependencies (yamllint, flake8, pyinstaller) are NOT managed by Dependabot and must be updated manually in `action.yml`.

## Flake8 Configuration

The action has a **configurable error threshold** for Flake8:

```bash
# Current threshold: 4 errors/warnings maximum
--max-line-length=150
--exclude=client/,.venv/,venv/
```

Pipeline fails if error count > 4 (action.yml:76). This threshold may need adjustment when adding strict linting rules.

## PyInstaller Binary Building

Binary building is **conditional** and requires:
1. Push event to a tag (`refs/tags/*`)
2. Non-empty `pyinstaller-binary-name` input
3. A pip project (uv projects fail with an error)

The binary is automatically attached to GitHub releases (action.yml:86-111).

## Action Inputs

Required inputs when using this action:

| Input | Required | Purpose |
|-------|----------|---------|
| `token` | Yes | GitHub token used to attach binaries to releases |
| `package-manager` | No | `auto` (default), `uv` or `pip` |
| `linter` | No | `ruff`, `flake8` or `none`; default depends on package manager |
| `type-checker` / `type-check-args` | No | `mypy`, `pyright` or `none` (default) |
| `test-paths` / `pytest-args` | No | Switch pip projects to plain pytest; `pytest-args` goes through `PYTEST_ADDOPTS` |
| `working-directory` | No | Project directory for monorepos (default `.`) |
| `pyinstaller-binary-name` | No | If set, builds and releases a binary |
| `pyinstaller-entrypoint` | No | Script to build (default `main.py`) |

## Important Workflow Notes

- Projects using this action must have a `.python-version` file to specify Python version
- `requirements.txt` is optional - only installed if present (pip projects)
- Legacy pip tests only run if `import pytest` is found in root-level Python files
- Security scanning uses severity cutoff of "high"
