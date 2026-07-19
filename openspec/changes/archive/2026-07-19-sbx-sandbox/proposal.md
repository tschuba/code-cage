## Why

AI coding agents like Claude Code or pi need `--dangerously-skip-permissions` (or equivalent) to run autonomously — but without containment they have full access to the host filesystem including credentials (`~/.ssh`, `~/.aws`) and sensitive data. code-cage provides a microVM-isolated sandbox where agents can run unrestricted without touching the host.

## What Changes

- New project `code-cage` providing a sandboxed execution environment for Claude Code
- Install and configure Docker `sbx` CLI (no Docker Desktop required)
- Kit-based sandbox config: `~/.claude/` read-only mount, `ANTHROPIC_API_KEY` credential, MCP via `static-mcp`
- `cage claude` wrapper launches Claude Code with `--dangerously-skip-permissions` inside the sandbox

## Capabilities

### New Capabilities

- `sbx-environment`: Install, configure, and launch the Docker sbx sandbox with correct isolation settings
- `workspace-mount`: Mount only the target project directory into the sandbox, excluding host home and credentials
- `agent-launch`: `cage claude` entry point launching Claude Code inside the sandbox via kit

### Modified Capabilities

## Impact

- New project directory: `/Users/thomas/Projects/code-cage`
- Runtime dependency: `sbx` CLI (installed via Homebrew, requires Docker account for `sbx login`)
- No Docker Desktop required
- macOS 26 / Apple Silicon only (for now)
- Network: outbound allowed (required for agent API calls), no inbound
