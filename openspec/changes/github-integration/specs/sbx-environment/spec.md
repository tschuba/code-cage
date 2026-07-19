## MODIFIED Requirements

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
- **WHEN** the sandbox starts with the kit applied and `GH_TOKEN` is set on the host
- **THEN** outbound HTTPS traffic to `github.com`, `api.github.com`, `objects.githubusercontent.com`, `codeload.github.com`, and `raw.githubusercontent.com` is permitted

#### Scenario: Kit forwards GitHub token as sandbox env var
- **WHEN** `GH_TOKEN` is present in the host environment at launch time
- **THEN** it is available as the `GH_TOKEN` environment variable inside the sandbox, and the outbound proxy injects `Authorization: Bearer <token>` for requests to GitHub service domains
