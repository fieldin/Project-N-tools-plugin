---
name: version-release
description: >-
  Full firmware/bootloader release: bumps version in MongoDB, builds binaries,
  uploads to S3, notifies Slack. Use when the user says "release version",
  "version release", "publish release", "cut release", "do a release", or any variant.
---

# project-n-tools:version-release

Run a full Project N version release using `project_tools/version_release_cli.py`.

## Project Root Check

Before running any command, walk up from CWD looking for `project_tools/version_release_cli.py`.
If not found, stop and print:
```
Error: Not inside Project_N repo. Navigate to the Project_N root directory and retry.
```
Set the directory where `project_tools/version_release_cli.py` was found as `PROJECT_ROOT`. Run all commands from `PROJECT_ROOT`.

## Pre-flight Checklist

Print this before collecting parameters:
```
Pre-flight:
  [ ] project_tools/.env contains PROJECT_N_MONGO_URI
  [ ] project_tools/.env contains AWS_PROFILE=PowerUserAccess-838148646721
  [ ] project_tools/s3.py exists (SSO-based, no hardcoded credentials)
  [ ] AWS SSO: will auto-trigger 'aws sso login --profile PowerUserAccess-838148646721'
      if token is expired (browser opens — authorize there)
```

**AWS S3 uses SSO — no credentials file.** `project_tools/s3.py` reads `AWS_PROFILE`
from the environment / `.env` and creates a boto3 session with that profile.
If the SSO token is expired, the CLI auto-runs `aws sso login` before uploading.

## Parameters

Collect these from the user's message, or ask if missing:

| Parameter | Values | Required |
|-----------|--------|---------|
| `targets` | `SML`, `FX`, or both | yes |
| `mode` | `firmware`, `bootloader`, `both` | yes |
| `dry_run` | yes/no | yes — always ask if not mentioned |
| `mark_release` | yes/no | yes — always ask if not mentioned |
| `manual_version` | `major.minor.build` | no (omit for auto-increment) |
| `username` | string | yes — infer from `git config user.name`, or ask |
| `comment` | string | no (default: empty) |

**Inferring from message:**
- "SML" / "FX" in message → targets (can be both)
- "firmware" / "bootloader" / "both" → mode
- "dry run" / "dry-run" → dry_run=yes
- "mark release" / "mark as release" / "new minor" → mark_release=yes

### Mark-release (`-R`) rules

- `--mark-release` changes versioning to: **minor + 1, build = 0** (instead of build + 1).
- **Scope is determined by `--mode`** — the CLI has no separate scope flag:
  - `mode=firmware` → only firmware minor bumps
  - `mode=bootloader` → only bootloader minor bumps
  - `mode=both` → **both** firmware and bootloader minor bump independently
- To bump minor for only one when releasing both, run two separate commands with `mode=firmware` and `mode=bootloader`.

**Getting username:**
```bash
git config user.name
```
Use the output as `--username`. If empty, ask the user.

## Workflow: always dry-run first

Before executing a real release, run with `--dry-run` to show the user the exact versions that will be written. The dry run is fast (no build) and shows current → next version for each target. Only proceed to the real release after user confirms.

## Command

```bash
cd <PROJECT_ROOT>
python3 -m project_tools.version_release_cli \
  --targets <comma-separated e.g. SML,FX> \
  --mode <firmware|bootloader|both> \
  --username "<USERNAME>" \
  [--comment "<COMMENT>"] \
  [--dry-run] \
  [--mark-release] \
  [--manual-version <major.minor.build>]
```

**Supported flags (verified against CLI):**
- `--targets` — comma-separated: `SML`, `FX`, or `SML,FX`
- `--mode` — `firmware`, `bootloader`, or `both`
- `--username` — required
- `--dry-run` — simulate without writing to Mongo/S3/Slack
- `--mark-release` / `-R` — minor bump instead of build bump
- `--manual-version` — override auto-increment (e.g. `1.2.3`)
- `--comment` — optional release note

**`--mark-release-scope` does NOT exist in the CLI** (GUI only). Use separate `--mode` runs instead.

## Execution

1. Echo the full command before executing (mask nothing — no secrets in args).
2. **Dry run first** — confirm versions with user before the real release.
3. Run real release with Bash tool, timeout=1800000 (30 min — includes full firmware build).
4. Stream output — do not suppress.
5. On completion report:
   - ✓ or ✗
   - Version string released (visible in log output)
   - S3 upload confirmation (visible in log output)
   - Any errors verbatim

## Error Handling

- Exit code 1: print stderr and stop. Do not retry.
- "PROJECT_N_MONGO_URI" not set: tell user to add it to `project_tools/.env`.
- `ModuleNotFoundError: No module named 'project_tools.s3'`: `project_tools/s3.py` is missing — recreate it (SSO-based, reads `AWS_PROFILE` from env).
- AWS SSO browser opens: inform user to complete login in browser, then re-run.
- Do not retry automatically.
