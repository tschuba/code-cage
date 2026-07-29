## Why

Using `--dangerously-skip-permissions` without code-cage remains more convenient than code-cage because the cage blocks read access to files outside the project and prevents launching host apps like VS Code. This friction causes users to bypass the sandbox entirely, defeating its purpose. The security model is already sound (sbx proxy-managed credentials, microVM isolation, write-scoped to project); this change adds convenience without weakening it.

## What Changes

- **Read access expanded**: cage mounts `~/Downloads`, `~/Documents`, `~/Desktop`, and `$TMPDIR` read-only, so Claude can reference files outside the project without write risk; credential paths excluded by omission
- **Host app launch bridge**: a lightweight host daemon (`cage-opend`) watches a shared request directory; inside the cage, a `code` wrapper writes launch requests that the daemon fulfils on the host
- **Clipboard output moves to `$TMPDIR`**: `cage-clipd` writes to `$TMPDIR/cage-clipboard-<timestamp>-<name>.<ext>` instead of `~/.cage/clipboard/latest.*`; files are auto-cleaned by the OS, multiple clipboard events coexist with readable names, and the same mount (`$TMPDIR:ro`) serves both clipboard files and pi's native clipboard path

## Capabilities

### New Capabilities
- `read-mounts`: Common home subdirs and `$TMPDIR` mounted read-only; credential dirs excluded by not being listed
- `host-app-launch`: Bridge for cage-to-host app launch requests (VS Code and similar), following the same daemon pattern as `cage-clipd`

### Modified Capabilities
- `workspace-mount`: Requirement changes from "only project dir mounted" to "project dir rw + explicit read-only mounts; credential paths always excluded"
- `clipboard-bridge`: Output path changes from `~/.cage/clipboard/latest.*` to `$TMPDIR/cage-clipboard-<timestamp>-<name>.<ext>`; mount changes from `~/.cage/clipboard:ro` to `$TMPDIR:ro`

## Impact

- `cage` launch script: replaces `~/.cage/clipboard:ro` with `$TMPDIR:ro`; adds `~/Downloads:ro`, `~/Documents:ro`, `~/Desktop:ro`
- `cage-clipd`: output path and filename format change; `clean_outdir()` removed (OS handles cleanup)
- New `cage-opend` daemon script (Python, same pattern as `cage-clipd`)
- New `code` wrapper script installed inside the sandbox (via claude-kit install command)
- `install` script: starts `cage-opend` alongside `cage-clipd`
- WezTerm keybinding: `.current` pointer moves to `$TMPDIR/cage-clipboard.current`
