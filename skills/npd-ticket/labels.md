# NPD labels

Use only these labels, so they stay consistent (`firmware`, never `fw` or `FW`). Pick every label
that matches the area the code check touched; recommend them, the person confirms.

| Label | Use for |
|---|---|
| `firmware` | Project_N application code (FXN / FX / SML targets) |
| `bootloader` | Project_N_bootloader, or the bootloader binary/flash layout |
| `bench` | Work that needs or changes the bench device (FXN_6) setup |
| `mqtt` | MQTT topics, broker, frag protocol over MQTT |
| `serial` | Serial / frag protocol over serial, the bench pty |
| `config-tool` | Fieldin_config_tool |
| `server` | logs_receiver, fld-gateway, n-project-monitor (server side) |
| `docs` | Documentation-only changes |
| `tooling` | project_tools scripts, build, flash and release tooling, plugins |

## Adding a label

Edit this file in a PR to Project-N-tools-plugin (new row: label and what it is for). Never invent a
label inside a ticket.
