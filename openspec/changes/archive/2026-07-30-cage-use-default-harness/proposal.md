## Why

Running `cage` without arguments always starts the `claude` harness. Users who primarily use a different harness (e.g. `pi`) must type it every time, which is friction for the common case.

## What Changes

- `cage use <harness>` subcommand writes the user's preferred default harness to `~/.cage/config`
- `cage` with no harness argument reads `~/.cage/config` and falls back to `claude` if unset

## Capabilities

### New Capabilities
- `cage-default-harness`: User can set and persist a global default harness via `cage use <harness>`

### Modified Capabilities

## Impact

- `cage` script: reads config on startup; branches on `use` subcommand
- `~/.cage/config`: new file (created by `cage use`, read by `cage`)
- No changes to kit specs, mounts, or agent behavior
