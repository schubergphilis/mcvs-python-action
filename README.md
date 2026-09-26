# MCVS-python-action

[![GitHub release](https://img.shields.io/github/v/release/schubergphilis/mcvs-python-action)](https://github.com/schubergphilis/mcvs-python-action/releases)
[![License](https://img.shields.io/github/license/schubergphilis/mcvs-python-action)](LICENSE)

<img src="./assets/logos/mcvs-python-action.png" width="250">

Mission Critical Vulnerability Scanner (MCVS) Python Action. Create Python code without high and critical vulnerabilities.

## Usage

Create a `.github/workflows/python.yml` file with the following content:

```yaml
---
name: Python
"on": push
permissions:
  contents: read # write if pyinstaller-binary-name is non-empty
jobs:
  MCVS-python-action:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@some-hash # v4.2.2
        with:
          persist-credentials: false
      - uses: schubergphilis/mcvs-python-action@some-hash # v0.3.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

<!-- markdownlint-disable MD013 -->

| Option                  | Default | Required | Description                                                                                                                                                    |
| :---------------------- | :------ | -------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| linter                  |         |          | `ruff`, `flake8` or `none`. Defaults to ruff for uv projects and flake8 for pip projects. ruff runs with the project's own configuration and any finding fails |
| package-manager         | auto    |          | `auto`, `uv` or `pip`. `auto` uses uv (`uv sync --all-packages --frozen`) when `uv.lock` exists, else pip with `requirements.txt`                              |
| pyinstaller-binary-name |         |          | If populated, then a binary will be created using pyinstaller and attached to a release. pip projects only                                                     |
| pyinstaller-entrypoint  | main.py |          | The Python script that pyinstaller turns into a binary                                                                                                         |
| pytest-args             |         |          | Extra pytest arguments, e.g. `-m "not integration"`. Quoting works as in a shell                                                                               |
| test-paths              |         |          | Paths to test, e.g. `packages/*/tests`. Defaults to pytest's own discovery                                                                                     |
| token                   |         | x        | GitHub token used to attach binaries to releases                                                                                                               |
| type-check-args         |         |          | Arguments for the type checker, e.g. `--strict packages/*/src`. Globs are expanded                                                                             |
| type-checker            | none    |          | `mypy`, `pyright` or `none`                                                                                                                                    |
| working-directory       | .       |          | Directory of the Python project, for monorepos. For a uv workspace this is the workspace root                                                                  |

<!-- markdownlint-enable MD013 -->

`persist-credentials: false` keeps the token out of `.git/config`, so
third-party packages installed from `requirements.txt` cannot read it. The
action receives the token only through the `token` input.

Define the Python version of the project by adding it to a `.python-version`
file.

ruff, mypy, pyright and pytest run from the project environment
(`uv run` for uv, `python3 -m` for pip), so add the ones you use to the
project's dependencies, e.g. the `dev` dependency group.

### uv workspace example

```yaml
      - uses: schubergphilis/mcvs-python-action@some-hash # v0.3.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          type-checker: mypy
          type-check-args: --strict packages/*/src
          test-paths: packages/*/tests
          pytest-args: -m "not integration and not live" --import-mode=importlib
```

## Migrating to v0.3.0

pip projects need no changes. The action detects them because there is no
`uv.lock`, and it keeps installing `requirements.txt`, running `test.py`
with coverage of `main` and linting with flake8.

- **uv projects** (a `uv.lock` is present) are now installed with
  `uv sync --all-packages --frozen`, linted with ruff and tested with plain
  `pytest`. Set `package-manager: pip` to keep the old behaviour.
- **pip projects that set `test-paths` or `pytest-args`** run plain `pytest`
  instead of `test.py`. Pass `--cov` options in `pytest-args` if you need
  coverage.
- **PyInstaller** binaries are now named after `pyinstaller-binary-name`.
  Before, they were always built as `gomod-go-version-updater`, and the
  release upload failed for any other name. Set `pyinstaller-entrypoint` if
  your script is not `main.py`.
