## Why

Cage sessions have no project-level configuration — mounts and network domains are hardcoded in kit specs or the launch script, with no way for a project (or the user globally) to declare its own requirements. There is also no persistent shared workspace: the agent can't retain context across session restarts, and sharing files with the agent beyond the clipboard requires no established convention.

## What Changes

- A `.cage/` directory in the project root is auto-created at cage launch, serving as the project-level cage workspace
- A three-level config system controls mounts and allowed network domains:
  - `~/.cage/config.yaml` — global user config (mounts + domains, not committed)
  - `PROJECT/.cage/config.yaml` — committed project config (network domains only; paths are machine-specific so mounts don't belong here)
  - `PROJECT/.cage/config.local.yaml` — gitignored project-local config (mounts for this project on this machine)
- Configs merge additively: global → project → local; kit domains are the floor for network
- A new `cage-config` Python helper parses and merges configs, feeding results into the `cage` launch script
- `PROJECT/.cage/share/` is established as the bidirectional project-scoped file sharing space: the user drops reference files there; the agent writes outputs there. This is already accessible via the existing project dir mount — the addition is convention and CLAUDE.md documentation
- `PROJECT/.cage/scratch/` is the agent's persistent workspace, surviving session restarts
- `PROJECT/.cage/.gitignore` is auto-created to commit `config.yaml` while ignoring `config.local.yaml`, `scratch/`, and `share/`
- The existing clipboard bridge (`~/.cage/clipboard/`) remains global and unchanged — it serves a different purpose (reactive, transient, daemon-managed) and is not merged into `share/`

## Capabilities

### New Capabilities

- `project-cage-config`: Per-project and global declarative configuration for cage mounts and allowed network domains
- `project-cage-workspace`: A structured `.cage/` directory in the project root providing persistent agent scratch space and a deliberate file-sharing channel

### Modified Capabilities

- `workspace-mount`: Extended — additional mounts from `~/.cage/config.yaml` and `PROJECT/.cage/config.local.yaml` are now included in the sbx invocation
- `cage-kit-install`: Extended — allowed network domains from global and project configs are merged into the effective kit at launch time

## Impact

- `cage`: updated to auto-create `.cage/`, call `cage-config`, and build extended sbx args
- `cage-config`: new Python helper script (alongside `cage` and `cage-clipd`)
- `install`: link `cage-config` onto PATH
- `claude-kit/spec.yaml`: no change (domains from configs are injected at launch, not into the kit source)
- `README.md`: document `.cage/` directory, config files, and share/scratch conventions
- CLAUDE.md (agent-facing): document `share/` and `scratch/` conventions for the agent
