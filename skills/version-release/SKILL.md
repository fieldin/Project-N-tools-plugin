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
| `mark_release_scope` | `firmware`, `bootloader`, `both` | conditional (see rules below) |
| `manual_version` | `major.minor.build` | no (omit for auto-increment) |
| `username` | string | yes — infer from `git config user.name`, or ask |
| `comment` | string | no (default: empty) |

**Inferring from message:**
- "SML" / "FX" in message → targets (can be both)
- "firmware" / "bootloader" / "both" → mode
- "dry run" / "dry-run" → dry_run=yes
- "mark release" / "mark as release" → mark_release=yes
- "both versions", "both app and bootloader" → mark_release_scope=both
- "app only", "firmware only" → mark_release_scope=firmware
- "bootloader only" → mark_release_scope=bootloader

### Mark-release scope rules

- `-R` / `--mark-release` changes versioning to: **minor + 1, build = 0**.
- Default scope is **firmware only** (`mark_release_scope=firmware`).
- If user asks for mark-release and `mode=both`, explicitly confirm whether scope is:
  - both firmware+bootloader,
  - firmware only,
  - bootloader only.
- If user does not explicitly request a scope, use **firmware only**.
- If `mode=firmware`, scope is firmware.
- If `mode=bootloader`, scope is bootloader.

**Getting username:**
```bash
git config user.name
```
Use the output as `--username`. If empty, ask the user.

## Command

```bash
cd <PROJECT_ROOT>
python3 -m project_tools.version_release_cli \
  --targets <comma-separated e.g. SML,FX> \
  --mode <firmware|bootloader|both> \
  --username "<USERNAME>" \
  [--comment "<COMMENT>"] \
  [--dry-run] \
  [-R|--mark-release] \
  [--mark-release-scope <firmware|bootloader|both>] \
  [--manual-version <major.minor.build>]
```

## Execution

1. Echo the full command before executing (mask nothing — no secrets in args).
2. Run with Bash tool, timeout=1800000 (30 min — builds take time).
3. Stream output — do not suppress.
4. On completion report:
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
