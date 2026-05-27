---
name: flash
description: >-
  Flash a compiled Project_N binary to a connected device via pyocd.
  Default is full image (bootloader + header + application) — same as the
  mbed-tools VSCode extension's "flash" button. Also supports flashing
  application-only, bootloader-only, or device id when explicitly requested.
  Use when the user says "flash", "flash app", "flash application",
  "flash bootloader", "flash device id", "flash firmware", or any variant.
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

| Parameter | Values | Required for |
|-----------|--------|--------------|
| `target` | `SML`, `FX` | always |
| `profile` | `develop`, `release` | always |
| `what` | `all` (default), `application`, `bootloader`, `device-id` | always |

**Default behavior: `what=all`** — flashes the full merged image (`Project_N.bin` = bootloader + header + application) at `0x08000000`. This matches the mbed-tools VSCode extension's flash button exactly:

1. Erase the flash-flag sector (`0x081C0000` SML / `0x080F0000` FX)
2. Write `0x01` at `FLASH_FLAG_ADDR` — signals "freshly flashed" to the bootloader (distinguishes a fresh flash from an OTA update)
3. Load `Project_N.bin` with `--connect under-reset` at frequency `1800000`

The flag byte is read by the bootloader on boot (`main.cpp:1481`) and logged as `Flashed flag: 1`. Device ID region is preserved.

**Inferring `what` from message:**
- "flash" / "flash firmware" / "flash all" / "flash full" / no explicit target → what=all (default)
- "flash app only" / "flash application only" / "app-only" → what=application
  - User MUST explicitly say "application" or "app only" — bare "flash" is NOT application
- "flash bootloader" / "bootloader only" → what=bootloader
- "flash device id" / "flash device-id" → what=device-id

**Inferring target/profile:**
- "SML" / "FX" in message → target
- "release" / "develop" in message → profile (default: develop)

If `what` is ambiguous (e.g. user previously did app-only build and now says just "flash"), ask: "Flash full image (bootloader + header + app) or application-only?"

## Commands

```bash
# flash all (DEFAULT) — full merged image at 0x08000000
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py flash-all -m <TARGET> -p <PROFILE> -t GCC_ARM

# flash application only (writes app + dummy header at bootloader-end addr)
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py flash-application -m <TARGET> -p <PROFILE> -t GCC_ARM

# flash bootloader only (at 0x08000000)
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py flash-bootloader -m <TARGET> -p <PROFILE> -t GCC_ARM

# flash device id
cd <PROJECT_ROOT>
python3 project_tools/build_tools.py flash-device-id -m <TARGET> -p <PROFILE> -t GCC_ARM
```

## Execution

1. Echo the command to be run before executing.
2. For `flash-all`, verify `cmake_build/<TARGET>/<PROFILE>/GCC_ARM/Project_N.bin` exists. If missing, tell user to run `build-application` first.
3. Ensure a device is connected — pyocd will fail immediately if none found.
4. Run with Bash tool (default timeout).
5. On completion report ✓ or ✗ and any errors verbatim.

## Error Handling

- If `build_tools.py` exits non-zero, print the full output and stop.
- If pyocd reports "No connected debug probes", tell user to connect device and retry.
- If `flash-all` fails with "Full image not found", tell user to compile firmware first (`project-n-tools:compile`).
- Do not retry automatically.
