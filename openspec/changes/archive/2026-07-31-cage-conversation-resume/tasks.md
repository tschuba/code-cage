## 1. cage Script

- [x] 1.1 Add `mkdir -p "$HOME/.cage/claude-projects"` before the `sbx run` line (alongside the existing `mkdir -p "$HOME/.cage/open"`)
- [x] 1.2 Add `"$HOME/.cage/claude-projects"` as a rw mount argument to both `sbx run` lines in `cage`

## 2. Kit Install Command

- [x] 2.1 In `claude-kit/spec.yaml` install command: after the existing symlink loop, derive `CAGE_PROJECTS` from `SRC` and symlink `/home/agent/.claude/projects` → `$CAGE_PROJECTS`

## 3. Verification

- [x] 3.1 Start cage, have a short exchange with Claude Code, exit the cage — confirm `~/.cage/claude-projects/` contains history files on the host
- [x] 3.2 Start a new cage session in the same directory, run `claude --resume` — confirm it picks up the prior conversation
- [x] 3.3 Start two cage sessions in different project dirs — confirm both write to `~/.cage/claude-projects/` without conflict
- [ ] 3.4 Run `openspec apply` to merge the `cage-conversation-persistence` spec into `openspec/specs/`
