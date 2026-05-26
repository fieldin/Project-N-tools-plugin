---
name: compile
description: >-
  Compile Project_N firmware for SML or FX targets using build_tools.py.
  Use when the user says "compile", "build", "compile firmware", "compile bootloader",
  "compile all", "build SML", "build FX", "build release", "build develop", or any
  variant of building/compiling the firmware or bootloader.
---

# project-n-tools:compile

Compile firmware using `project_tools/build_tools.py`.

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
| `operation` | `firmware`, `bootloader`, `all` | always |
| `override` | yes/no | bootloader or all operations only |

**Inferring from message:**
- "compile firmware" / "build app" → operation=firmware
- "compile bootloader" → operation=bootloader
- "compile all" / "build all" / "bootloader+firmware" → operation=all
- "SML" / "FX" in message → target
- "release" / "develop" in message → profile
- If override not mentioned for bootloader ops → ask: "Override bootloader binary in `bootloader/` dir? (yes/no)"

## Commands

**firmware only:**
```bash
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py build_application -m <TARGET> -p <PROFILE> -t GCC_ARM
```

**bootloader only:**
```bash
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py build_bootloader -m <TARGET> -p <PROFILE> -t GCC_ARM [-o]
```
(`-o` = override flag, copies compiled bootloader bin into `bootloader/` dir)

**all (bootloader then firmware):**
Run bootloader command first, then firmware command. If bootloader fails, stop — do not run firmware.

## Execution

1. Echo the command(s) to be run before executing.
2. Run with Bash tool. For release builds set timeout=600000.
3. Stream output — do not suppress.
4. On completion report:
   - ✓ or ✗ per step
   - Output binary location (typically `cmake_build/<TARGET>/<PROFILE>/GCC_ARM/`)
   - Any errors verbatim

## Error Handling

- If `build_tools.py` exits non-zero, print the last 20 lines of output and stop.
- Do not retry automatically — report failure and let user decide.
- If `Project_N_bootloader/` subdir missing, warn: "Project_N_bootloader directory not found — bootloader compile will fail."
