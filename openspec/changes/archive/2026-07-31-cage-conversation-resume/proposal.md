## Why

Agent conversation history lives inside the ephemeral sandbox (`/home/agent/.claude/projects/`) and is discarded when the cage session ends. This makes `claude --resume` ineffective — there is nothing to resume. Users lose all conversation context every time they close a cage session.

## What Changes

- A persistent host directory (`~/.cage/claude-projects/`) is mounted into the sandbox read-write
- The install command symlinks `/home/agent/.claude/projects/` to that mount, so Claude Code writes conversation history to the host
- History survives sandbox teardown; `claude --resume` works in a subsequent cage session for the same project

## Capabilities

### New Capabilities
- `cage-conversation-persistence`: Conversation history written by Claude Code inside a cage session persists on the host and is available for resume in future sessions

### Modified Capabilities

## Impact

- `cage` script: add `mkdir -p "$HOME/.cage/claude-projects"` and the new mount argument to `sbx run`
- `claude-kit/spec.yaml` install command: add symlink for `projects`
- No effect on other mounts, kit specs, or agent behavior
- Default behavior remains "start fresh" — resume requires the user to explicitly pass `--resume` to claude
