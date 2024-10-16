# MCVS-python-action

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
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v4.1.1
      - uses: schubergphilis/mcvs-python-action@v0.1.1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

<!-- markdownlint-disable MD013 -->

| Option                  | Default                              | Required | Description                                                                                                       |
| :---------------------- | :----------------------------------- | -------- | :---------------------------------------------------------------------------------------------------------------- |
| pyinstaller-binary-name |                                      |          | If populated, then a binary will be created using pyinstaller and attached to a release                           |
| token                   | ' '                                  | x        | GitHub token that is required to push a package to the registry of the project and to pull cached Trivy DB images |
| trivy-action-db         | ghcr.io/aquasecurity/trivy-db:2      |          | Replace this with a cached image to prevent bump into pull rate limiting issues                                   |
| trivy-action-java-db    | ghcr.io/aquasecurity/trivy-java-db:1 |          | Replace this with a cached image to prevent bump into pull rate limiting issues                                   |

<!-- markdownlint-enable MD013 -->

Define the Python version of the project by adding it to a `.python-version`
file.
