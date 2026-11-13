<!-- markdownlint-disable -->

# Hardening Report: ariga--setup-atlas/v0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ariga--setup-atlas/v0.6** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Prepare Atlas Install' writes `version`, which comes from inputs.version through INPUT_VERSION, to GITHUB_OUTPUT with `echo "version=$version" >> "$GITHUB_OUTPUT"`. Newlines are not stripped first with printf '%s' | tr -d '\n\r', so a version containing a newline can add extra output keys. The `key=` line is safe because its substitution removes every character outside [A-Za-z0-9._-].

Locations:

- `action.yml:33`

### github-env-injection (severity: high)

Step 'Install Atlas' writes ATLAS_BIN_DIR to GITHUB_PATH with `echo "$ATLAS_BIN_DIR" >> "$GITHUB_PATH"`. ATLAS_BIN_DIR comes from steps.atlas-install.outputs.dir, which is built from the inherited $HOME. Newlines are not stripped with printf '%s' | tr -d '\n\r' before the write.

Locations:

- `action.yml:60`

### github-env-injection (severity: high)

Step 'Trim Token' writes the value of inputs.cloud-token (through RAW and TRIMMED) to GITHUB_OUTPUT with `echo "trimmed=$TRIMMED" >> $GITHUB_OUTPUT`. The value is only passed through xargs, not the printf '%s' | tr -d '\n\r' step, and the $GITHUB_OUTPUT path is unquoted.

Locations:

- `action.yml:93`

### github-env-injection (severity: high)

Step 'Expose Grant' writes TOKEN, which comes from steps.trim_token.outputs.trimmed and originally from inputs.cloud-token, to GITHUB_ENV with `echo "ATLAS_TOKEN=$TOKEN" >> $GITHUB_ENV`. There is no printf '%s' | tr -d '\n\r' step, so a value containing a newline could set other environment variables such as BASH_ENV.

Locations:

- `action.yml:116`

### script-injection (severity: high)

Rule (a): the Windows pwsh install step puts expressions directly inside its run: script, in `$version = "${{ inputs.version }}"` and `if ("${{ inputs.flavor }}" -ne "") {`. Attacker-controlled input is substituted into the PowerShell source before it runs. Pass these values through env: and read them as $env:INPUT_VERSION and $env:INPUT_FLAVOR instead.

Locations:

- `action.yml:76`
- `action.yml:77`

### unsafe-shell (severity: high)

Step 'Install Atlas' pipes a remote script straight into a shell: `curl -sSf https://atlasgo.sh | CI=true sh -s -- -o "$ATLAS_BIN_DIR/atlas"`. Download the installer to a file first, ideally check it, then run it.

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

All findings are fixed in action.yml. I haven't run the action or its tests after these edits.

- **Output and env writes:** before each value is written to GITHUB_OUTPUT, GITHUB_PATH or GITHUB_ENV, newlines are now removed with printf '%s' | tr -d '\n\r'. This covers the version, the bin directory, the trimmed token and ATLAS_TOKEN. The $GITHUB_OUTPUT and $GITHUB_ENV paths are now in double quotes where they weren't.
- **Installer:** the atlasgo.sh script is now downloaded to a temp file and run with `CI=true sh "$INSTALL_SCRIPT" -o ...`. I dropped the `-s --` from the old piped form so the script still gets `-o` as its first argument. The script isn't checked in any way before it runs (no checksum or signature).
- **Windows step:** inputs.version and inputs.flavor now go through env: and the PowerShell script reads them as $env:INPUT_VERSION and $env:INPUT_FLAVOR.

