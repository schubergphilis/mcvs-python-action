# MCVS-python-action

[![GitHub release](https://img.shields.io/github/v/release/schubergphilis/mcvs-python-action)](https://github.com/schubergphilis/mcvs-python-action/releases)
[![License](https://img.shields.io/github/license/schubergphilis/mcvs-python-action)](LICENSE)

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
      - uses: schubergphilis/mcvs-python-action@some-hash # v0.2.1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

<!-- markdownlint-disable MD013 -->

| Option                  | Default | Required | Description                                                                                                |
| :---------------------- | :------ | -------- | :--------------------------------------------------------------------------------------------------------- |
| pyinstaller-binary-name |         |          | If populated, then a binary will be created using pyinstaller and attached to a release                    |
| token                   |         | x        | GitHub token required for Docker registry authentication and uploading release assets (if building binary) |

<!-- markdownlint-enable MD013 -->

Define the Python version of the project by adding it to a `.python-version`
file.
