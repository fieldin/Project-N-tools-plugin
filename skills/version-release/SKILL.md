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
  [ ] AWS SSO: will auto-trigger 'aws sso login' if token expired (browser opens)
```

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
- "mark release" / "mark as release" → mark_release=yes

**Getting username:**
```bash
git config user.name
```
Use the output as `--username`. If empty, ask the user.

## Command

```bash
cd <PROJECT_ROOT>
python3 project_tools/version_release_cli.py \
  --targets <comma-separated e.g. SML,FX> \
  --mode <firmware|bootloader|both> \
  --username "<USERNAME>" \
  [--comment "<COMMENT>"] \
  [--dry-run] \
  [--mark-release] \
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
- AWS SSO browser opens: inform user to complete login in browser, then re-run.
- Do not retry automatically.
