# IBM Detect Secret scanner

> Originally forked from [secret-scanner/action](https://github.com/secret-scanner/action)

---

## The Problem

People will sometimes commit secrets to a GitHub repository

## How it works

Uses [`IBM/detect-secrets`](https://github.com/IBM/detect-secrets) to look for newly committed secrets. If it finds any potential secrets, it will:
* Fail
* Create a Job Summary with a list of the potential secrets found, and some advice on how to deal with the issue
* Provide an updated secrets baseline that contains the newly added secrets. This is useful if secrets that were discovered are not actually secrets.

## How to use it

### Installation

First, create a `.secrets.baseline` in the repo you want to add this action to. For more details on what this file represents, visit [the README for IBM/detect-secrets](https://github.com/IBM/detect-secrets#detect-secrets):

```bash
cd PATH_TO_REPOSITORY
pip install git+https://github.com/ibm/detect-secrets.git@<VERSION>
detect-secrets scan --update .secrets.baseline  .
detect-secrets audit .secrets.baseline
```

Second, add this GitHub action to your workflow or create a new one. A basic workflow would be:
```yaml
# File: .github/workflows/ibm-detect-secrets.yml
name: Checking for Secrets
on: push
jobs:
  check-secrets:
    name: Checking for Secrets
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
      - name: IBM Detect Secrets scanner
        uses: bigpick/ibm-detect-secrets-action@0.0.1
```

### Ignoring false positives

Often you might have strings that the secret scanner determines are secrets, but are actually harmless. Examples of these might be:

1. Docker tags
2. Git commit SHA's
3. Randomly generated base64 strings

For these cases, it is useful to ignore certain files, lines, or "secrets". You can do this using the files:

- `.github/actions/secret-scanner/excluded_files.patterns`
- `.github/actions/secret-scanner/excluded_secrets.patterns`
- `.github/actions/secret-scanner/excluded_lines.patterns`

While the path prefix defaults to `.github/actions/secret-scanner`, you can change this with the
input `exclude_files_path`. Blank lines and lines starting with `#` will be ignored.

#### How to use excluded_files.patterns

On each line, write the regex for the path to the file to ignore. For example:

```
# File: .github/actions/secret-scanner/excluded_files.patterns
# Lines starting with the char '#' are ignored
.*-sealed\.json$
\.github/actions/spelling/
```

will exclude files ending in `-sealed.json` and everything in the `.github/actions/spelling` folder

#### How to use excluded_secrets.patterns

On each line write the regex for a secret you wish to ignore. For example:

```
# File: .github/actions/secret-scanner/excluded_secrets.patterns
^SHA256:[A-Fa-f0-9]{64}
```

#### How to use excluded_lines.patterns

On each line write the regex for a line you with to ignore. For example:

```
# File: .github/actions/secret-scanner/excluded_lines.patterns
^\s+with\s+imageTag\s*=.*$
```

will exclude the line `  with imageTag = <ANY_STRING>`

#### How to do more advanced exclusions

You can also pass arguments to `detect-secrets` directly by using `detect-secret-additional-args`. For information on the arguments that you can pass, visit [IBM/detect-secrets#filters](https://github.com/IBM/detect-secrets#command-line). For example:

```yaml
name: Checking for Secrets
on: push
env:
  SCANNER_ARGS: |
      --exclude-files \.github/actions/spelling/.*
      --exclude-lines ^\s+with\s+imageTag\s*=.*$
jobs:
  check-secrets:
    name: Checking for Secrets
    runs-on: ubuntu-latest
      ...
        with:
          detect_secret_additional_args: ${{ env.SCANNER_ARGS }}
```

This will ignore everything in `.github/actions/spelling/*`, and any line that matches the regex `^\s+with\s+imageTag\s*=.*$`.

### Inputs

| Input                           | Description                                                                          | Required | default value                          |
| -----                           | -----------                                                                          | -------- | -------------                          |
| `detect_secrets_version`        | The version of IBM/detect-secrets to use                                             | no       | `0.13.1+ibm.62.dss`                    |
| `detect_secret_additional_args` | Extra arguments to pass to the `detect-secret` binary when it is looking for secrets | no       | No additional arguments (empty string) |
| `baseline_file`                 | A path to the baseline secrets file                                                  | no       | `.secrets.baseline`                    |
| `python_version`                | The version of python to use                                                         | no       | 3.11.9                                 |
| `exclude_files_path`            | A path to the direcotry files containing things to exclude                           | no       | `.github/actions/secret-scanner`       |
