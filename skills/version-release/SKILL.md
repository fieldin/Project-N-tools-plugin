---
name: version-release
description: >-
  Full firmware/bootloader release: bumps version in MongoDB, builds binaries,
  uploads to S3, notifies Slack. A bootloader release always co-releases firmware
  (lockstep). Use when the user says "release version", "version release",
  "publish release", "cut release", "do a release", or any variant.
---

# project-n-tools:version-release

Run a full Project N version release using `project_tools/version_release_cli.py`.

## Lockstep model (read first)

- Firmware and bootloader move together; everything tracks the latest bootloader.
- A bootloader release **always** ships a matching firmware: `--mode bootloader`
  is auto-promoted to `--mode both`. Releasing a bootloader alone is not possible
  by design.
- `--mode both` releases the bootloader, then stamps that exact bootloader version
  as the firmware's required-bootloader, so the pair cannot drift.
- Versions **auto-increment per kind** from each one's own history in MongoDB.
  **Never ask the operator for version numbers.**
- Bootloader and firmware are **each** independently a **bump** or a **release**:
  - release → minor bump + marked stable (`X.(Y+1).0`)
  - bump    → build increment (`X.Y.(Z+1)`)

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

Collect from the user's message, or ask if missing:

| Parameter | Values | Required |
|-----------|--------|----------|
| `targets` | `SML`, `FX`, `FXN`, or any combination | yes |
| `mode` | `firmware` (app only) or `both` (bootloader + firmware). `bootloader` is accepted but auto-promotes to `both`. | yes |
| `bootloader_release` | bump / release | ask whenever a bootloader is released (mode `both`) |
| `firmware_release` | bump / release | yes — always ask |
| `dry_run` | yes/no | yes — always ask if not mentioned |
| `username` | string | yes — infer from `git config user.name`, or ask |
| `comment` | string | no (default: empty) |

**Do NOT ask for version numbers** — they auto-increment per kind.

**Ask, in order, before running:**
1. Targets (if not given).
2. Mode: firmware-only, or both (releasing a new bootloader)? A bootloader release always becomes `both`.
3. If a bootloader is being released (`both`): is the **bootloader** a bump or a release?
4. Is the **firmware** a bump or a release?
5. Dry run?
6. Username (infer from git, confirm).

**Inferring from message:**
- "SML" / "FX" / "FXN" → targets (any combination)
- "firmware" / "app only" → mode=firmware; "bootloader" / "both" → mode=both
- "dry run" / "dry-run" → dry_run=yes
- "release" for a kind → that kind's flag = release; "bump" / "candidate" → that kind = bump

**Getting username:**
```bash
git config user.name
```
Use the output as `--username`. If empty, ask the user.

## Command

```bash
cd <PROJECT_ROOT>
python3 -m project_tools.version_release_cli \
  --targets <comma-separated e.g. SML,FX,FXN> \
  --mode <firmware|both> \
  --username "<USERNAME>" \
  [--comment "<COMMENT>"] \
  [--dry-run] \
  [--bootloader-release] \
  [--firmware-release]
```

- Add `--bootloader-release` only if the bootloader is a release (omit for a bump).
- Add `--firmware-release` only if the firmware is a release (omit for a bump).
- Omit both flags = both kinds are plain build bumps.
- `--mark-release` still exists as a shared shortcut (marks both as release); prefer the per-kind flags.

## Execution

1. Echo the full command before executing (mask nothing — no secrets in args).
2. Run with Bash tool, timeout=1800000 (30 min — builds take time).
3. Stream output — do not suppress.
4. On completion report:
   - ✓ or ✗
   - Version string released for each kind (visible in log output)
   - S3 upload confirmation (visible in log output)
   - Any errors verbatim

## Error Handling

- Exit code 1: print stderr and stop. Do not retry.
- "PROJECT_N_MONGO_URI" not set: tell user to add it to `project_tools/.env`.
- AWS SSO browser opens: inform user to complete login in browser, then re-run.
- Do not retry automatically.
