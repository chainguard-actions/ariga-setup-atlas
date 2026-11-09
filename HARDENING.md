<!-- markdownlint-disable -->

# Hardening Report: ariga--setup-atlas/v0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ariga--setup-atlas/v0** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Prepare Atlas Install': the value of inputs.version / inputs.flavor (passed in through INPUT_VERSION) is written to $GITHUB_OUTPUT without the printf '%s' | tr -d '\n\r' cleanup step: `echo "version=$version" >> "$GITHUB_OUTPUT"`. If the input contains a newline, an attacker can inject extra outputs.

Locations:

- `action.yml:33`

### github-env-injection (severity: high)

Step 'Install Atlas': ATLAS_BIN_DIR comes from steps.atlas-install.outputs.dir and is written to $GITHUB_PATH without the cleanup step: `echo "$ATLAS_BIN_DIR" >> "$GITHUB_PATH"`.

Locations:

- `action.yml:60`

### github-env-injection (severity: high)

Step 'Trim Token': RAW comes from inputs.cloud-token and is written to $GITHUB_OUTPUT after only an `echo | xargs` trim, not the required printf '%s' | tr -d '\n\r' cleanup: `echo "trimmed=$TRIMMED" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:93`

### github-env-injection (severity: high)

Step 'Expose Grant': TOKEN comes from steps.trim_token.outputs.trimmed, which traces back to inputs.cloud-token. It is written to $GITHUB_ENV without the cleanup step: `echo "ATLAS_TOKEN=$TOKEN" >> $GITHUB_ENV`. A token value containing a newline could set arbitrary environment variables, such as LD_PRELOAD or BASH_ENV, for later steps.

Locations:

- `action.yml:117`

### script-injection (severity: high)

Rule (a): the Windows pwsh step puts expressions straight into the script text: `$version = "${{ inputs.version }}"` and `if ("${{ inputs.flavor }}" -ne "") {`. A crafted input can break out of the string and run arbitrary PowerShell. These values should go through env: and be read as $env:INPUT_VERSION / $env:INPUT_FLAVOR.

Locations:

- `action.yml:76`
- `action.yml:77`

### unsafe-shell (severity: high)

Step 'Install Atlas' pipes remote content straight into a shell: `curl -sSf https://atlasgo.sh | CI=true sh -s -- -o "$ATLAS_BIN_DIR/atlas"`. The script should be downloaded to a file, checked, and then run as a separate command.

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

All findings were in action.yml. Changes:

- **Writes to $GITHUB_OUTPUT, $GITHUB_PATH and $GITHUB_ENV:** the version, bin dir, trimmed token and ATLAS_TOKEN values now go through `printf '%s' | tr -d '\n\r'` before they are written, so a newline in an input can't add extra outputs or environment variables.
- **Windows PowerShell step:** `inputs.version` and `inputs.flavor` are now passed through `env:` and read as `$env:INPUT_VERSION` and `$env:INPUT_FLAVOR` instead of being pasted into the script.
- **Atlas installer:** the script from atlasgo.sh is now downloaded to a temp file with `mktemp` and the step stops if the file is empty. It is then run with `CI=true sh "$INSTALL_SCRIPT" -o ...`. I dropped the `-s --` from the old pipe form so the installer still gets `-o` as its first argument. The empty-file test only catches a failed download; nothing verifies the script's contents.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml: (1) The Prepare step now strips newlines and carriage returns from RUNNER_OS and RUNNER_ARCH using printf '%s' | tr -d '\n\r' before building the cache key. (2) The atlas-cache step now strips newlines from $HOME/.atlas before writing it, and quotes $GITHUB_OUTPUT. (3) The Windows pwsh step removes CR and LF from $targetPath with -replace "[`r`n]", "" before Add-Content $env:GITHUB_PATH. The pwsh change uses PowerShell syntax because that step runs under pwsh, not bash. Nothing was run to test these changes.

