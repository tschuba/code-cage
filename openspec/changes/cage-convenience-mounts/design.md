## Context

code-cage uses sbx microVM sandboxes. The current `cage` script mounts exactly three paths into the sandbox: the working directory (rw), `~/.claude` (ro), and `~/.cage/clipboard` (ro). Everything else is inaccessible.

This is the right default, but it creates friction: Claude cannot read a file in `~/Downloads` unless the user copies it into the project, and `code .` typed inside the cage silently fails because the microVM has no connection to the host display or app layer.

The clipboard bridge (`cage-clipd`) already proves the pattern for host↔cage data sharing: a host daemon copies data into `~/.cage/clipboard/`, which is mounted read-only. The same approach applies to both new capabilities.

## Goals / Non-Goals

**Goals:**
- Allow Claude to read files outside the project (Downloads, configurable paths) without gaining write access
- Allow `code .` and similar host-app launches to work from inside the cage
- Keep credential paths (`~/.aws`, `~/.ssh`, `~/.gnupg`, `~/.netrc`) unmounted unconditionally
- Zero changes to the sbx secret/proxy model for credentials

**Non-Goals:**
- Write access to anything outside the project directory
- Mounting paths not explicitly listed (no "read the whole home dir")
- Generalised two-way IPC between cage and host beyond app launch

## Decisions

### 1. Read mounts: named home subdirs + `$TMPDIR` in `cage` script

Mount `~/Downloads`, `~/Documents`, `~/Desktop`, and `$TMPDIR` as read-only in the `cage` script. sbx has no exclusion-mount primitive, so "home dir ro minus credentials" is implemented as a whitelist of named subdirs; credential paths are excluded by not being listed.

`$TMPDIR` is included because: (a) cage-clipd now writes clipboard output there, and (b) pi's native clipboard path (`$TMPDIR/pi-clipboard-*.png`) becomes accessible without any additional mechanism.

A config file for user-customisable mounts was considered but rejected — YAGNI. Additional paths can be appended as `:ro` args to `sbx run`.

### 2. Host app launch: `cage-opend` daemon + shared socket directory

Follow the `cage-clipd` pattern exactly:
- Host daemon `cage-opend` watches `~/.cage/open/` for request files
- `~/.cage/open/` is mounted into the sandbox (write-only from cage side is sufficient, but sbx doesn't support write-only mounts, so mount rw — the daemon only reads from this dir, not writes)
- Inside the cage, a `code` wrapper script writes a JSON request file to `~/.cage/open/`; the daemon executes `open -a "Visual Studio Code" <path>` on the host and deletes the request file

**Why a file-based protocol over a socket?** The `sbx run` mount mechanism is directory-based. A socket inside a mounted directory works but requires polling or `inotify`; a simple file-drop + polling loop (like `cage-clipd`'s 0.3s interval) is sufficient for a UX that only needs ~1s response time.

**Why `open -a "Visual Studio Code"` instead of `code`?** `code` CLI may not be on the host `PATH` inside the daemon's environment. `open -a` is macOS-native and always works.

**Launcher whitelist:** the daemon only accepts `open-vscode` as the request type in v1. Adding more launchers later is a one-line addition to the daemon.

### 3. `code` wrapper installed via `claude-kit`

Add a `code` shell script to the kit's install commands, placed at `/usr/local/bin/code` inside the sandbox. It writes the request file and waits up to 2s for the daemon to consume it (signals success). This is transparent to Claude — `code .` works as expected.

### 4. Clipboard output: `$TMPDIR` with readable timestamped filenames

cage-clipd currently writes to `~/.cage/clipboard/latest.*` and calls `clean_outdir()` to remove stale files before each write. Problems: single `latest.*` filename loses history, requires explicit cleanup, and uses a cage-specific path that no other tool understands.

New approach: write to `$TMPDIR/cage-clipboard-<YYYYMMDD-HHMMSS>-<name>.<ext>`:
- Screenshot → `cage-clipboard-20260729-153045.png`
- PDF from clipboard → `cage-clipboard-20260729-153045.pdf`
- Finder file `report.pdf` → `cage-clipboard-20260729-153045-report.pdf`
- Multiple Finder files → one prefixed entry per file

`.current` pointer moves to `$TMPDIR/cage-clipboard.current` (same content: one path per line). WezTerm keybinding reads this file unchanged.

`clean_outdir()` is removed — macOS purges `$TMPDIR` on reboot and for inactive files. Multiple clipboard events coexist (distinguishable by name and timestamp), making it easy to verify which content will be referenced.

`~/.cage/clipboard/` dir and its mount are dropped entirely.

## Risks / Trade-offs

- **cage-opend not running**: `code` wrapper will timeout silently. Mitigation: `cage` script starts `cage-opend` alongside `cage-clipd` at launch.
- **$TMPDIR mount exposes all host temp files**: ephemeral by design, user-owned, no persistent secrets expected there. Acceptable for this threat model.
- **Request file races (two cage sessions)**: both write to the same `~/.cage/open/` dir. Mitigation: request filenames include the cage name (`<cage-name>-<timestamp>.json`), so they don't collide; daemon processes all files it finds.
- **$TMPDIR accumulates cage-clipboard-* files over a long session**: no manual cleanup, but files are small and macOS evicts them. Acceptable.

## Migration Plan

1. Add `cage-opend` script (no user action needed beyond re-running `install`)
2. Update `cage` script to add `~/Downloads:ro` mount and start `cage-opend`
3. Update `claude-kit/spec.yaml` install commands to drop the `code` wrapper
4. Existing cage sessions are unaffected until relaunched

No rollback needed — removing `~/Downloads:ro` from the `cage` script reverts to previous behaviour.
