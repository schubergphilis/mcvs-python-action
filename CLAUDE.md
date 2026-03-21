# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **composite GitHub Action** (not a standalone application) that provides the Mission Critical Vulnerability Scanner (MCVS) for Python projects. The action performs security scanning, linting, testing, and optional binary building for Python codebases.

## Architecture

### Composite Action Structure

The action is defined in `action.yml` and executes as a series of composite steps:

1. **YAML Linting**: Validates YAML files using yamllint
2. **Python Environment Setup**: Installs Python version from `.python-version`
3. **Security Scanning**: Uses Anchore scan-action to detect vulnerabilities
4. **Dependency Installation**: Installs packages from `requirements.txt` if present
5. **Testing**: Runs pytest if tests are detected
6. **Code Linting**: Uses Flake8 with a configurable error threshold
7. **Binary Building**: Conditionally builds PyInstaller binaries on tag releases

### Key Design Decisions

- **Composite vs Docker**: Uses `using: composite` to avoid Docker overhead and enable caching
- **Conditional Execution**: Steps like testing and binary building only run when applicable
- **Token Authentication**: Requires GitHub token for package registry and Docker registry access
- **Version Pinning**: All tools are pinned to specific versions for reproducibility

## Version Constraints

**CRITICAL**: The following versions are pinned in `action.yml`:

- `yamllint==1.37.1` (action.yml:21)
- `actions/setup-python@v5.6.0` (action.yml:28)
- `anchore/scan-action@v6.2.0` (action.yml:34)
- `flake8==7.2.0` (action.yml:75)
- `pyinstaller==v6.13.0` (action.yml:102)
- `svenstaro/upload-release-action@2.9.0` (action.yml:106)

When updating dependencies:
- Update the version in `action.yml`
- Dependabot automatically creates PRs for GitHub Actions updates (see `.github/dependabot.yml`)
- Python package versions must be updated manually

## Testing Changes

This action is tested via PR validation:

```yaml
# Validation happens automatically on PRs via .github/workflows/mcvs-pr-validation.yml
# Uses schubergphilis/mcvs-pr-validation-action@v0.2.0
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

Pipeline fails if error count > 4 (action.yml:81-83). This threshold may need adjustment when adding strict linting rules.

## PyInstaller Binary Building

Binary building is **conditional** and requires:
1. Push event to a tag (`refs/tags/*`)
2. Non-empty `pyinstaller-binary-name` input

The binary is automatically attached to GitHub releases (action.yml:89-111).

## Action Inputs

Required inputs when using this action:

| Input | Required | Purpose |
|-------|----------|---------|
| `token` | Yes | GitHub token for package registry and Docker login |
| `pyinstaller-binary-name` | No | If set, builds and releases a binary |

## Important Workflow Notes

- Projects using this action must have a `.python-version` file to specify Python version
- `requirements.txt` is optional - only installed if present
- Tests only run if `import pytest` is found in Python files
- Security scanning uses severity cutoff of "high" (action.yml:39)
- Docker login required for security scanning (action.yml:40-44)
