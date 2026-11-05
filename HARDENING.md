<!-- markdownlint-disable -->

# Hardening Report: ariga--setup-atlas/v0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ariga--setup-atlas/v0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in the bash install step, the inputs are inserted into the run: script directly: `version="${{ inputs.version }}"` and `if [ -n "${{ inputs.flavor }}" ]; then`. Rule (b): the value derived from the input is later expanded unquoted in `ATLAS_VERSION=$version CI=true sh`. Fix: pass the inputs through env: and double-quote every expansion.

Locations:

- `action.yml:24`
- `action.yml:25`
- `action.yml:28`

### script-injection (severity: high)

Rule (a): in the Windows pwsh step, the inputs are inserted into the run: script directly: `$version = "${{ inputs.version }}"` and `if ("${{ inputs.flavor }}" -ne "") {`. Fix: pass them through env: and read them as $env:VAR.

Locations:

- `action.yml:32`
- `action.yml:33`

### unsafe-shell (severity: high)

A remote script is piped straight into a shell: `curl -sSf https://atlasgo.sh | ATLAS_VERSION=$version CI=true sh`. Download the script to a file (ideally checking it), then run it as a separate step.

Locations:

- `action.yml:28`

### github-env-injection (severity: high)

The 'Trim Token' step writes a value derived from inputs.cloud-token (passed in through env RAW) to $GITHUB_OUTPUT: `TRIMMED=$(echo "$RAW" | xargs)` then `echo "trimmed=$TRIMMED" >> $GITHUB_OUTPUT`. The required `printf '%s' ... | tr -d '\n\r'` step is not applied before the write.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Expose Grant' step writes steps.trim_token.outputs.trimmed (passed in through env TOKEN) to $GITHUB_ENV without sanitizing it: `echo "ATLAS_TOKEN=$TOKEN" >> $GITHUB_ENV`. A newline in the value could inject extra environment variables. Apply `printf '%s' "$TOKEN" | tr -d '\n\r'` before the write.

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

**Fixes applied:** script-injection, unsafe-shell, github-env-injection, static-inline-injection

**Notes:**

Changes to action.yml (not run or tested):
- **Script injection (both install steps):** `inputs.version` and `inputs.flavor` now come in through `env:` as INPUT_VERSION and INPUT_FLAVOR. The bash step reads them as quoted shell variables; the Windows step reads them as `$env:INPUT_VERSION` and `$env:INPUT_FLAVOR`.
- **Unsafe shell:** the bash step no longer pipes atlasgo.sh into `sh`. It downloads the script to a temp file from `mktemp`, stops if the file is empty, runs it with `ATLAS_VERSION="$version" CI=true sh "$INSTALL_SCRIPT"`, then deletes it. The empty-file check is the only verification; the script's contents aren't checked against a checksum.
- **Env/output injection:** the 'Trim Token' step now strips newlines (`printf '%s' | tr -d '\n\r'`) before writing to `$GITHUB_OUTPUT`, and so does the 'Expose Grant' step before writing to `$GITHUB_ENV`. The file paths in both writes are now quoted.

Upstream's `xargs` whitespace trim on the token is unchanged.

### Iteration 2

**Fixes applied:** script-injection, unsafe-shell, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.version and inputs.flavor into env: in both the bash and pwsh steps. In bash they are read as $INPUT_*, in pwsh as $env:INPUT_*, and every expansion is quoted. The atlasgo.sh installer is now saved to a mktemp file and run as 'sh "$file"' with ATLAS_VERSION and CI set the same way as before. Upstream never ran the installer with errexit, so it still runs without it. Newlines are now stripped from $HOME before the atlas-cache step writes it to $GITHUB_OUTPUT, and that redirect is quoted. CR and LF are now stripped from the Windows target path before it is written to $GITHUB_PATH. None of these changes have been run yet.

