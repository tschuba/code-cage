## Why

Using `--dangerously-skip-permissions` without code-cage remains more convenient than code-cage because the cage blocks read access to files outside the project and prevents launching host apps like VS Code. This friction causes users to bypass the sandbox entirely, defeating its purpose. The security model is already sound (sbx proxy-managed credentials, microVM isolation, write-scoped to project); this change adds convenience without weakening it.

## What Changes

- **Read access expanded**: cage mounts `~/Downloads` and a configurable list of additional host paths as read-only, so Claude can reference files outside the project without write risk
- **Credential paths excluded**: `~/.aws`, `~/.ssh`, `~/.gnupg`, `~/.netrc`, and similar paths remain unmounted even when broader read access is granted
- **Host app launch bridge**: a lightweight host daemon (`cage-opend`) watches a shared socket directory; inside the cage, a `code` wrapper writes launch requests that the daemon fulfils on the host
- **`workspace-mount` spec updated**: relaxes the "project directory only" requirement to allow explicit read-only additions

## Capabilities

### New Capabilities
- `read-mounts`: Additional host paths mounted read-only into the sandbox, with an explicit credential-path exclusion list
- `host-app-launch`: Bridge for cage-to-host app launch requests (VS Code and similar), following the same daemon pattern as `cage-clipd`

### Modified Capabilities
- `workspace-mount`: Requirement changes from "only project dir mounted" to "project dir rw + explicit read-only mounts; credential paths always excluded"

## Impact

- `cage` launch script: adds ro mount arguments for Downloads and any user-configured paths
- `cage-kit/spec.yaml`: no changes needed (env var isolation already handled by sbx proxy)
- New `cage-opend` daemon script (Python, same pattern as `cage-clipd`)
- New `code` wrapper script installed inside the sandbox (via claude-kit install command)
- `install` script: starts `cage-opend` alongside `cage-clipd`
