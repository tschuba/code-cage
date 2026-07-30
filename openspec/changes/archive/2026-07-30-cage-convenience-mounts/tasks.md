## 1. Read Mounts

- [x] 1.1 Add `"$HOME/Downloads:ro" "$HOME/Documents:ro" "$HOME/Desktop:ro" "$TMPDIR:ro"` to `sbx run` in the `cage` script; remove `"$HOME/.cage/clipboard:ro"`
- [x] 1.2 Verify cage cannot write to any of those dirs (`touch ~/Downloads/.cage-test` inside sandbox → permission error)
- [x] 1.3 Verify credential paths inaccessible (`cat ~/.aws/credentials` inside sandbox → file-not-found)
- [x] 1.4 Verify `$TMPDIR/cage-clipboard.current` is readable inside the cage

## 2. Host App Launch Daemon

- [x] 2.1 Create `cage-opend` Python script: polls `~/.cage/open/` every 0.3s, reads `*.json` request files, runs `open -a "Visual Studio Code" <path>`, deletes the request file
- [x] 2.2 Add `cage-opend` startup to `cage` script (same pattern as `cage-clipd`: check `pgrep`, start in background, create `~/.cage/open/` dir)
- [x] 2.3 Add `"$HOME/.cage/open"` mount to `sbx run` in the `cage` script (rw, so cage can write request files); add kit install step to symlink `/home/agent/.cage/open → <host>/.cage/open` (sbx does not support source:dest remapping — mounts at same absolute path)

## 3. `code` Wrapper Inside Cage

- [x] 3.1 Add install command to `claude-kit/spec.yaml` that writes `/usr/local/bin/code` inside the sandbox: writes a JSON request file to `~/.cage/open/<cage-name>-<timestamp>.json` and polls for deletion (up to 2s) to confirm launch
- [x] 3.2 Verify `code .` from inside a running cage opens VS Code on the host

## 4. Clipboard Bridge Migration

- [x] 4.1 Update `cage-clipd`: change output paths to `$TMPDIR/cage-clipboard-<YYYYMMDD-HHMMSS>[-<name>].<ext>`; update `.current` write to `$TMPDIR/cage-clipboard.current`; remove `clean_outdir()` and `OUTDIR` setup
- [x] 4.2 Update WezTerm keybinding: read `$TMPDIR/cage-clipboard.current` instead of `~/.cage/clipboard/.current`
- [x] 4.3 Verify: screenshot → `$TMPDIR/cage-clipboard-<timestamp>.png` exists and `.current` points to it
- [x] 4.4 Verify: Finder file copy → `$TMPDIR/cage-clipboard-<timestamp>-<name>` and `.current` updated
- [x] 4.5 Verify: second clipboard event leaves prior file in place (no cleanup)

## 5. Spec Sync

- [x] 5.1 Run `openspec apply` to merge delta specs into `openspec/specs/`
