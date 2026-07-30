## Why

Starting `cage` in a directory that already has a running session kills it: the script uses a deterministic name (`agent-projectdir`) and runs `sbx rm -f "$NAME"` unconditionally before starting. This makes it impossible to open a second terminal in the same project without destroying the first session.

## What Changes

- Session names gain a PID suffix (`agent-projectdir-$$`) so each invocation is unique
- `sbx rm -f "$NAME"` is removed — nothing to clean up when names are unique
- Multiple cage instances for the same project (or different projects with the same directory basename) can now run simultaneously

## Capabilities

### New Capabilities
- `cage-parallel-sessions`: Multiple cage instances for the same project can run concurrently without interfering with each other

### Modified Capabilities

## Impact

- `cage` script: two-line change (name generation + remove rm line)
- No effect on kit specs, mounts, or agent behavior
- Tab title and prompt identity remain unchanged (`⬡ cage:<agent>`)
