# Repos the NPD agents work in

GitHub org: `fieldin`. The code check reads every repo at **`origin/<base>`**, the branch the agents
branch from, never the local clone's current branch.

| Repo | Base branch | Agents may |
|---|---|---|
| `Project_N` | `fxn_integration` | change (branch, commit, PR) |
| `Project_N_bootloader` | `fxn_integration` | change (always asks for plan approval first) |
| `FX-bringup` | `master` | change |
| `n-project-monitor` | `master` | read only (test harness) |
| `logger_client` | `main` | read only (Project_N submodule) |
| `Fieldin_config_tool` | `master` | not yet (later phase): file the ticket, agents won't pick it up |
| `logs_receiver` | `main` | not yet (later phase) |
| `fld-gateway` | `main` | not yet (later phase) |

Any other repo (e.g. `Project_N_zephyr`, which is out of scope): warn the person that the agents
won't work there before writing the ticket.

## Reading code at the base branch

With a local clone: `git -C <clone> fetch origin <base>` then `git -C <clone> show origin/<base>:<path>`
and `git -C <clone> grep -n <pattern> origin/<base> -- <path>`.
Without one: `gh api repos/fieldin/<repo>/contents/<path>?ref=<base>` (base64 `content`) or
`gh search code --repo fieldin/<repo> <pattern>`.
