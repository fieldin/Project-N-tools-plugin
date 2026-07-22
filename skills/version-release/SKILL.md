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
  **Never ask the operator for version numbers** — `--manual-version` exists as an
  override but is not the normal path.
- Bootloader and firmware are **each** independently a **bump** or a **release**,
  via `--bootloader-release` / `--firmware-release`:
  - release → minor bump + marked stable (`X.(Y+1).0`)
  - bump    → build increment (`X.Y.(Z+1)`)
- The old shared `--mark-release` / `-R` still exists (marks both kinds as release
  when set, both as bump when unset) — prefer the per-kind flags so you can e.g.
  release the bootloader while only bumping firmware.

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
|-----------|--------|----------|
| `targets` | `SML`, `FX`, `FXN`, or any comma-separated combination | yes |
| `mode` | `firmware` (app only) or `both` (bootloader + firmware). `bootloader` is accepted but auto-promotes to `both`. | yes |
| `bootloader_release` | bump / release | ask whenever a bootloader is released (mode `both`) |
| `firmware_release` | bump / release | yes — always ask |
| `dry_run` | yes/no | yes — always ask if not mentioned |
| `username` | string | yes — infer from `git config user.name`, or ask |
| `comment` | string | no (default: empty) |

**Do NOT ask for version numbers** — they auto-increment per kind. `--manual-version`
is an override for the rare case the user explicitly asks for one; don't offer it.

**Ask, in order, before running:**
1. Targets (if not given).
2. Mode: firmware-only, or both (releasing a new bootloader)? A bootloader release always becomes `both`.
3. If a bootloader is being released (`both`): is the **bootloader** a bump or a release?
4. Is the **firmware** a bump or a release?
5. Dry run?
6. Username (infer from git, confirm).

**Inferring from message:**
- "SML" / "FX" / "FXN" in message → targets (can be multiple)
- "firmware" / "app only" → mode=firmware; "bootloader" / "both" → mode=both (bootloader always promotes to both)
- "dry run" / "dry-run" → dry_run=yes
- "release" for a kind, "mark release", "new minor" → that kind's flag = release; "bump" / "candidate" → that kind = bump

**Getting username:**
```bash
git config user.name
```
Use the output as `--username`. If empty, ask the user.

## Workflow: always dry-run first

Before executing a real release, run with `--dry-run` to show the user the exact versions that will be written. The dry run is fast (no build for `mode=firmware`; still builds for a bootloader/both release, but skips all Mongo writes and S3 uploads) and shows current → next version for each target and kind. Only proceed to the real release after the user confirms the versions shown.

## Command

```bash
cd <PROJECT_ROOT>
python3 -m project_tools.version_release_cli \
  --targets <comma-separated e.g. SML,FX,FXN> \
  --mode <firmware|bootloader|both> \
  --username "<USERNAME>" \
  [--comment "<COMMENT>"] \
  [--dry-run] \
  [--bootloader-release] \
  [--firmware-release] \
  [--manual-version <major.minor.build>]
```

**Supported flags (verified against CLI):**
- `--targets` — comma-separated: `SML`, `FX`, `FXN`, or combinations such as `SML,FX,FXN`
- `--mode` — `firmware`, `bootloader` (auto-promoted to `both`), or `both`
- `--username` — required
- `--dry-run` — simulate without writing to Mongo/S3/Slack
- `--bootloader-release` — bootloader is a release (minor bump + stable); omit for a build bump
- `--firmware-release` — firmware is a release (minor bump + stable); omit for a build bump
- `--mark-release` / `-R` — shared shortcut: minor bump instead of build bump for both kinds (per-kind flags override; prefer them)
- `--manual-version` — override auto-increment (e.g. `1.2.3`) — rare, only if explicitly requested
- `--comment` — optional release note

**`--mark-release-scope` does NOT exist in the CLI** (GUI only). Use `--bootloader-release` / `--firmware-release` instead.

## Execution

1. Echo the full command before executing (mask nothing — no secrets in args).
2. **Dry run first** — confirm versions with user before the real release.
3. Run real release with Bash tool, timeout=1800000 (30 min — includes full firmware build).
4. Stream output — do not suppress.
5. On completion report:
   - ✓ or ✗
   - Version string released for each kind (visible in log output)
   - S3 upload confirmation (visible in log output)
   - Any errors verbatim

## Error Handling

- Exit code 1: print stderr and stop. Do not retry.
- "PROJECT_N_MONGO_URI" not set: tell user to add it to `project_tools/.env`.
- `ModuleNotFoundError: No module named 'project_tools.s3'`: `project_tools/s3.py` is missing — recreate it (SSO-based, reads `AWS_PROFILE` from env).
- AWS SSO browser opens: inform user to complete login in browser, then re-run.
- Do not retry automatically.
