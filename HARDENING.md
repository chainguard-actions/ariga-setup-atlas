<!-- markdownlint-disable -->

# Hardening Report: ariga--setup-atlas/v0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ariga--setup-atlas/v0.5** was hardened automatically. 8 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Prepare Atlas Install' writes `echo "version=$version" >> "$GITHUB_OUTPUT"`. `$version` comes from INPUT_VERSION (inputs.version) and INPUT_FLAVOR, and is never cleaned with printf '%s' | tr -d '\n\r' first. A newline in the input could inject extra outputs.

Locations:

- `action.yml:33`

### github-env-injection (severity: high)

Step 'Install Atlas' writes `echo "$ATLAS_BIN_DIR" >> "$GITHUB_PATH"` without the printf '%s' | tr -d '\n\r' cleanup. ATLAS_BIN_DIR comes from steps.atlas-install.outputs.dir, which counts as untrusted under this check.

Locations:

- `action.yml:60`

### github-env-injection (severity: high)

Step 'Trim Token' writes `echo "trimmed=$TRIMMED" >> $GITHUB_OUTPUT`. TRIMMED comes from inputs.cloud-token through `echo "$RAW" | xargs`, which is not the required printf '%s' | tr -d '\n\r' cleanup. $GITHUB_OUTPUT is also left unquoted.

Locations:

- `action.yml:93`

### github-env-injection (severity: high)

Step 'Expose Grant' writes `echo "ATLAS_TOKEN=$TOKEN" >> $GITHUB_ENV`. TOKEN comes from steps.trim_token.outputs.trimmed, which traces back to inputs.cloud-token. It is not cleaned with printf '%s' | tr -d '\n\r', so newlines could inject arbitrary environment variables.

Locations:

- `action.yml:117`

### script-injection (severity: high)

Sub-rule (a): the Windows pwsh step puts expressions straight into the script: `$version = "${{ inputs.version }}"` and `if ("${{ inputs.flavor }}" -ne "") {`. An attacker-controlled input can break out of the string and run arbitrary PowerShell. Pass these through env: and read them as $env:INPUT_VERSION / $env:INPUT_FLAVOR instead.

Locations:

- `action.yml:76`
- `action.yml:77`

### unsafe-shell (severity: high)

Step 'Install Atlas' pipes remote content straight into a shell: `curl -sSf https://atlasgo.sh | CI=true sh -s -- -o "$ATLAS_BIN_DIR/atlas"`. Download the script to a file, check it, then run it as a separate step.

Locations:

- `action.yml:59`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:76`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flavor }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:77`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, unsafe-shell, static-inline-injection

**Notes:**

Strips newlines with printf '%s' | tr -d '\n\r' before each write to $GITHUB_OUTPUT, $GITHUB_PATH and $GITHUB_ENV: the version, the bin dir, the trimmed token and ATLAS_TOKEN. Quotes $GITHUB_OUTPUT and $GITHUB_ENV. The atlasgo.sh installer is now downloaded to a temp file, checked to be non-empty, then run as `CI=true sh "$INSTALL_SCRIPT" -o ...` (the '-s --' is dropped). The Windows pwsh step now passes inputs.version and inputs.flavor through env and reads them as $env:INPUT_VERSION and $env:INPUT_FLAVOR.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Prepare Atlas Install: version is sanitized with printf | tr -d '\n\r' before being written to GITHUB_OUTPUT (cache key derives from sanitized value too). Expose Grant: token sanitized immediately before writing to GITHUB_ENV, and "$GITHUB_ENV" quoted.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed in action.yml, not run or re-scanned yet: (1) The 'Expose Grant' step now strips newlines from TOKEN with printf|tr right before writing it to $GITHUB_ENV. The $GITHUB_ENV path is now quoted. (2) The 'Prepare Atlas Install' step strips newlines from HOME, RUNNER_OS and RUNNER_ARCH before writing dir and key to $GITHUB_OUTPUT. (3) The atlas-cache step strips newlines from HOME before writing path to $GITHUB_OUTPUT. (4) The Windows pwsh step removes CR/LF from $targetPath before Add-Content to $GITHUB_PATH.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

In the Windows pwsh step of action.yml, $targetPath now has CR and LF removed ($safePath = $targetPath -replace "[`r`n]", '') before Add-Content writes it to GITHUB_PATH. The directory creation and the download are unchanged.

