## Why

When context-switching between a cage session and a regular host terminal, there is no visual signal distinguishing the two — the shell prompt looks identical. Users lose track of which terminal is inside the microVM and which is on the host.

## What Changes

- Each cage kit injects a `CAGE_AGENT` env var and a distinctive shell prompt + terminal title into the container's `.bashrc` at startup
- The prompt prefix and tab title make cage sessions immediately recognizable at a glance, both at the shell and while an agent is running

## Capabilities

### New Capabilities

- `cage-session-identity`: Shell prompt and terminal title reflect the cage agent (claude/pi) inside every cage session, with no indicator on the host

### Modified Capabilities

- `cage-kit-install`: Existing kit install commands gain an additional step that writes the identity env var and prompt customization to `/home/agent/.bashrc`

## Impact

- `claude-kit/spec.yaml`: one new install command
- `pi-kit/spec.yaml`: one new install command
- No host-side changes, no new dependencies
