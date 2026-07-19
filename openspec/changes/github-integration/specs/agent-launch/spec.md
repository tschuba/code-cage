## MODIFIED Requirements

### Requirement: Claude Code can be launched inside the sandbox via kit
The system SHALL provide a `cage claude` entry point that applies the `claude-kit`, starts Claude Code with `--dangerously-skip-permissions`, and injects GitHub credentials and git identity when available on the host.

#### Scenario: Standard launch
- **WHEN** the user runs `cage claude` from a project directory
- **THEN** `sbx run claude --kit <code-cage>/claude-kit -- --dangerously-skip-permissions` executes, the current directory is mounted as workspace, `~/.claude/` is available read-only, and `ANTHROPIC_API_KEY` is injected via the kit

#### Scenario: Skills and plugins are available
- **WHEN** Claude Code starts inside the sandbox
- **THEN** it loads skills (ponytail, superpowers, context7) from the kit-mounted `~/.claude/` exactly as it would on the host

#### Scenario: GitHub token injected when host is authenticated
- **WHEN** the user runs `cage claude` and `gh auth token` succeeds on the host
- **THEN** the cage script exports `GH_TOKEN` from `gh auth token` before launching sbx, and the kit forwards it into the sandbox

#### Scenario: gitconfig mounted when present on host
- **WHEN** `~/.gitconfig` exists on the host and `GH_TOKEN` is available
- **THEN** `~/.gitconfig` is mounted read-only into the sandbox so commits use the host user's name and email

#### Scenario: Launch succeeds without GitHub auth
- **WHEN** `gh auth token` returns empty or is not installed on the host
- **THEN** cage skips token injection and gitconfig mount, and sbx launches normally with local-only git capabilities
