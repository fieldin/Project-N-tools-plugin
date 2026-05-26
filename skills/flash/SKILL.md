---
name: flash
description: >-
  Flash a compiled Project_N binary to a connected device via pyocd.
  Use when the user says "flash app", "flash application", "flash bootloader",
  "flash device id", "flash firmware", or any variant of flashing to a device.
---

# project-n-tools:flash

Flash a built binary to a connected device using `project_tools/build_tools.py`.

## Project Root Check

Before running any command, walk up from CWD looking for `project_tools/build_tools.py`.
If not found, stop and print:
```
Error: Not inside Project_N repo. Navigate to the Project_N root directory and retry.
```
Set the directory where `project_tools/build_tools.py` was found as `PROJECT_ROOT`. Run all commands from `PROJECT_ROOT`.

## Parameters

Collect these from the user's message, or ask if missing:

| Parameter | Values | Required for |
|-----------|--------|--------------|
| `target` | `SML`, `FX` | always |
| `profile` | `develop`, `release` | always |
| `what` | `application`, `bootloader`, `device-id` | always |

**Inferring from message:**
- "flash app" / "flash application" / "flash firmware" → what=application
- "flash bootloader" → what=bootloader
- "flash device id" / "flash device-id" → what=device-id
- "SML" / "FX" in message → target
- "release" / "develop" in message → profile (default: develop if not specified)

## Commands

```bash
# flash application
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py flash_application -m <TARGET> -p <PROFILE> -t GCC_ARM

# flash bootloader
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py flash_bootloader -m <TARGET> -p <PROFILE> -t GCC_ARM

# flash device id
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py flash_device_id -m <TARGET> -p <PROFILE> -t GCC_ARM
```

## Execution

1. Echo the command to be run before executing.
2. Ensure a device is connected — pyocd will fail immediately if none found.
3. Run with Bash tool (default timeout).
4. On completion report ✓ or ✗ and any errors verbatim.

## Error Handling

- If `build_tools.py` exits non-zero, print the full output and stop.
- If pyocd reports "No connected debug probes", tell user to connect device and retry.
- Do not retry automatically.
