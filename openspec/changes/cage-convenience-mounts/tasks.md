## 1. Read Mounts

- [ ] 1.1 Add `"$HOME/Downloads:ro" "$HOME/Documents:ro" "$HOME/Desktop:ro"` mount arguments to `sbx run` in the `cage` script
- [ ] 1.2 Verify the cage cannot write to any of those dirs (test: `touch ~/Downloads/.cage-test` inside sandbox, expect permission error)
- [ ] 1.3 Verify credential paths remain inaccessible (test: `cat ~/.aws/credentials` and `cat ~/.ssh/id_rsa` inside sandbox, expect file-not-found)

## 2. Host App Launch Daemon

- [ ] 2.1 Create `cage-opend` Python script: polls `~/.cage/open/` every 0.3s, reads `*.json` request files, runs `open -a "Visual Studio Code" <path>`, deletes the request file
- [ ] 2.2 Add `cage-opend` startup to `cage` script (same pattern as `cage-clipd`: check `pgrep`, start in background, create `~/.cage/open/` dir)
- [ ] 2.3 Add `"$HOME/.cage/open"` mount to `sbx run` in the `cage` script (rw, so cage can write request files)

## 3. `code` Wrapper Inside Cage

- [ ] 3.1 Add install command to `claude-kit/spec.yaml` that writes `/usr/local/bin/code` inside the sandbox: writes a JSON request file to `~/.cage/open/<cage-name>-<timestamp>.json` and polls for deletion (up to 2s) to confirm launch
- [ ] 3.2 Verify `code .` from inside a running cage opens VS Code on the host

## 4. Spec Sync

- [ ] 4.1 Run `openspec apply` to merge delta specs into `openspec/specs/`
