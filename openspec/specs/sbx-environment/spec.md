## ADDED Requirements

### Requirement: sbx CLI is installed and authenticated
The system SHALL have the `sbx` CLI installed via Homebrew and authenticated with a Docker account before any sandbox can be launched.

#### Scenario: Fresh install
- **WHEN** the user runs the setup script on a machine without sbx
- **THEN** sbx is installed via `brew install docker/tap/sbx` and `sbx login` is triggered for OAuth authentication

#### Scenario: Already installed
- **WHEN** sbx is already installed and authenticated
- **THEN** setup script skips install and confirms readiness without re-authenticating

### Requirement: Sandbox provides microVM-level isolation
The sandbox runtime SHALL use Docker sbx's microVM isolation so that each agent session has its own filesystem, Docker daemon, and network stack independent from the host.

#### Scenario: Agent cannot access host home directory
- **WHEN** an agent is running inside the sandbox
- **THEN** paths outside the mounted workspace and explicitly declared kit mounts (e.g., `~/.ssh`, `~/.aws`, `~/Documents`) are not accessible

#### Scenario: Sandbox is ephemeral
- **WHEN** an agent session ends (the sandbox exits)
- **THEN** all state installed inside the sandbox (packages, temp files, daemon cache) is discarded

### Requirement: A kit provides the complete sandbox configuration
The system SHALL use an sbx Kit (`spec.yaml`) as the single source of truth for sandbox configuration, covering `~/.claude/` access, credentials, MCP server definitions, and GitHub network access.

#### Scenario: Kit mounts ~/.claude/ read-only
- **WHEN** the sandbox starts with the kit applied
- **THEN** `~/.claude/` from the host is available inside the sandbox at the same absolute path, read-only, so Claude Code finds its skills, plugins, and CLAUDE.md without any manual sync

#### Scenario: Kit injects API credentials
- **WHEN** the sandbox starts with the kit applied
- **THEN** `ANTHROPIC_API_KEY` (and other required keys) are available as environment variables inside the sandbox, sourced from the host environment at launch time

#### Scenario: Kit defines MCP servers
- **WHEN** the sandbox starts with the kit applied
- **THEN** MCP servers (e.g., context7) are available to the agent via the kit's `static-mcp` configuration, without requiring a separate MCP server process on the host

#### Scenario: Kit allows outbound HTTPS to GitHub
- **WHEN** the sandbox starts with the kit applied
- **THEN** outbound HTTPS traffic to `github.com`, `api.github.com`, `objects.githubusercontent.com`, `codeload.github.com`, and `raw.githubusercontent.com` is permitted

#### Scenario: GitHub token available via sbx secret mechanism
- **WHEN** a GitHub token has been stored with `sbx secret set -g github` on the host
- **THEN** sbx sets `GH_TOKEN` inside the sandbox (via the proxy-managed secret mechanism), and the outbound proxy authenticates GitHub API and git HTTPS requests using the stored token — the raw token is never directly accessible inside the sandbox
