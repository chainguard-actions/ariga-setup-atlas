<!-- markdownlint-disable -->

# Hardening Report: ariga--setup-atlas/v0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ariga--setup-atlas/v0.3** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the Bash step at line 23 puts expressions directly into the shell script: `version="${{ inputs.version }}"` (line 24) and `if [ -n "${{ inputs.flavor }}" ]` (line 25). Rule (b): `$version` is unquoted at line 28: `ATLAS_VERSION=$version CI=true sh`.

Locations:

- `action.yml:24`
- `action.yml:25`
- `action.yml:28`

### script-injection (severity: high)

Rule (a): the PowerShell step at line 31 puts expressions directly into the script: `$version = "${{ inputs.version }}"` (line 32) and `if ("${{ inputs.flavor }}" -ne "")` (line 33).

Locations:

- `action.yml:32`
- `action.yml:33`

### script-injection (severity: high)

Rule (a): the 'Login to Atlas' step puts an expression directly into the script, without quotes: `atlas login --token ${{ inputs.cloud-token }}`.

Locations:

- `action.yml:44`

### unsafe-shell (severity: high)

Remote content is piped straight into a shell: `curl -sSf https://atlasgo.sh | ATLAS_VERSION=$version CI=true sh`. The script should be downloaded to a file first and then run as a separate step.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

The Windows step writes `$targetPath` to GITHUB_PATH: `Add-Content $env:GITHUB_PATH "$targetPath"`. `$targetPath` is built from `$env:LocalAppData`. That variable is inherited from the process environment, not set by this script, and nothing strips newlines or carriage returns before the write.

Locations:

- `action.yml:40`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:24`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flavor }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:25`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:32`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flavor }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:33`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cloud-token }}" appears directly in run: block of step "Login to Atlas"; move to env: map

Locations:

- `action.yml:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unsafe-shell, github-env-injection

**Notes:**

Bash step: inputs.version and inputs.flavor now come in through env as INPUT_VERSION and INPUT_FLAVOR. The installer from atlasgo.sh is saved to a mktemp file and run with ATLAS_VERSION="$version" CI=true sh "$file". The piped form had no '--', so there was nothing to drop. The temp file is deleted afterwards. Pwsh step: the inputs are read as $env:INPUT_*. $targetPath has CR and LF removed before it is written to GITHUB_PATH. Login step: the token comes in through env as ATLAS_CLOUD_TOKEN and is passed double-quoted. The script-injection finding also flagged $version as unquoted on line 28; it is now quoted in the new run line. None of these changes were run or tested.

