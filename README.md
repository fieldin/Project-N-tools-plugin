# project-n-tools

Claude Code / Cursor plugin for Project N firmware operations.

## Installation

```bash
# One-time: add the marketplace
claude plugin marketplace add https://github.com/fieldin/Project-N-tools-plugin

# Then install
claude plugin install project-n-tools@Project-N-tools-plugin
```

## Skills

| Skill | Trigger examples |
|-------|-----------------|
| `project-n-tools:compile` | "compile firmware SML develop", "build bootloader FXN release" |
| `project-n-tools:flash` | "flash application SML", "flash bootloader FXN release" |
| `project-n-tools:device-id` | "create device id fxn_test_1 FXN develop" |
| `project-n-tools:version-release` | "release version firmware SML FX FXN" |

## Requirements

- All skills must be run from inside the `Project_N` repo root.
- `version-release` requires `project_tools/.env` with `PROJECT_N_MONGO_URI` set.
- `version-release` requires AWS SSO configured (`aws sso login` runs automatically if token expired).
