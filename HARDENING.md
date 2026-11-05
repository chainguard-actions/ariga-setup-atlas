<!-- markdownlint-disable -->

# Hardening Report: ariga--setup-atlas/v0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ariga--setup-atlas/v0.4** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

(a) The bash install step puts expressions straight into the run: script: `version="${{ inputs.version }}"` and `if [ -n "${{ inputs.flavor }}" ]; then`. (b) The same step also expands the input-derived variable without quotes: `curl -sSf https://atlasgo.sh | ATLAS_VERSION=$version CI=true sh`.

Locations:

- `action.yml:24`
- `action.yml:25`
- `action.yml:28`

### script-injection (severity: high)

(a) The Windows pwsh install step puts expressions straight into the run: script: `$version = "${{ inputs.version }}"` and `if ("${{ inputs.flavor }}" -ne "") {`.

Locations:

- `action.yml:32`
- `action.yml:33`

### unsafe-shell (severity: high)

Remote content goes straight into a shell: `curl -sSf https://atlasgo.sh | ATLAS_VERSION=$version CI=true sh`. The script should be downloaded to a file and checked before it is run.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

The 'Trim Token' step takes inputs.cloud-token (through env RAW and `echo "$RAW" | xargs`) and writes it to $GITHUB_OUTPUT with `echo "trimmed=$TRIMMED" >> $GITHUB_OUTPUT`. The required `printf '%s' ... | tr -d '\n\r'` sanitization is missing.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Expose Grant' step writes steps.trim_token.outputs.trimmed (through env TOKEN) to $GITHUB_ENV with `echo "ATLAS_TOKEN=$TOKEN" >> $GITHUB_ENV`. The required `printf '%s' "$TOKEN" | tr -d '\n\r'` sanitization is missing.

Locations:

- `action.yml:73`

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

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unsafe-shell, github-env-injection

**Notes:**

All changes are in action.yml. Nothing was run or tested.

- **Expressions in run: scripts:** In both install steps, `inputs.version` and `inputs.flavor` now come in through `env:` (INPUT_VERSION, INPUT_FLAVOR). The bash step reads them as `$VAR` and the pwsh step as `$env:VAR`.
- **Unquoted variable:** `$version` is now quoted when passed to the installer.
- **Piping curl into sh:** The bash step now saves the atlasgo.sh script to a temp file made with mktemp and stops if the file is empty. Then it runs `ATLAS_VERSION="$version" CI=true sh "$INSTALL_SCRIPT"` and deletes the file. That empty-file check is the only check: the script's contents and signature are not verified.
- **Trim Token:** The `echo | xargs` trim is kept as it was. Newlines are then stripped with `printf '%s' | tr -d '\n\r'` before the value goes to `$GITHUB_OUTPUT`.
- **Expose Grant:** The token has its newlines stripped the same way before it is written to `$GITHUB_ENV`. Both of these output paths are now quoted.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

I fixed all three places in action.yml where the action writes to GitHub's special files. Nothing was run afterwards to test the changes.

1. **Expose Grant:** `TOKEN` now has newlines stripped (`printf '%s' | tr -d '\n\r'`) right before it is written to `$GITHUB_ENV`, and `$GITHUB_ENV` is now quoted.
2. **Windows install step:** `$targetPath` now has carriage returns and newlines removed with `-replace` right before `Add-Content $env:GITHUB_PATH`.
3. **atlas-cache:** `$HOME/.atlas` now has newlines stripped right before it is written to `$GITHUB_OUTPUT`, and `$GITHUB_OUTPUT` is now quoted.

None of these changes alter what the action does for normal values: the token and the two paths only lose newline characters, which they shouldn't contain anyway.

