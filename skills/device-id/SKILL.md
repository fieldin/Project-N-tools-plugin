---
name: device-id
description: >-
  Create a device ID binary for a Project_N device and flash it.
  Use when the user says "create device id", "generate device id",
  "make device id", "assign device id", or any variant.
---

# project-n-tools:device-id

Create a device ID binary using `project_tools/build_tools.py`.

## Project Root Check

Before running any command, walk up from CWD looking for `project_tools/build_tools.py`.
If not found, stop and print:
```
Error: Not inside Project_N repo. Navigate to the Project_N root directory and retry.
```
Set the directory where `project_tools/build_tools.py` was found as `PROJECT_ROOT`. Run all commands from `PROJECT_ROOT`.

## Parameters

Collect these from the user's message, or ask if missing:

| Parameter | Values | Required |
|-----------|--------|---------|
| `target` | `SML`, `FX` | always |
| `profile` | `develop`, `release` | always |
| `device_id` | string e.g. `sml_test_1` | always |

**Inferring from message:**
- "SML" / "FX" in message → target
- "release" / "develop" in message → profile (default: develop if not specified)
- Any quoted string or word after "device id" / "device-id" → device_id
- If device_id not clear, ask: "What device ID string should I use? (e.g. sml_test_1)"

## Command

`create_device_id` prompts interactively for the device ID string. Pass it via stdin:

```bash
cd <PROJECT_ROOT>
echo "<DEVICE_ID>" | python3 project_tools/build_tools.py create_device_id -m <TARGET> -p <PROFILE> -t GCC_ARM
```

## Execution

1. Echo the command (with device_id shown) before executing.
2. Run with Bash tool (default timeout).
3. On completion report ✓ or ✗ and output binary location.
4. Any errors verbatim.

## Error Handling

- If `build_tools.py` exits non-zero, print the full output and stop.
- Do not retry automatically.
