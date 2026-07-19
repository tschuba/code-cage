## ADDED Requirements

### Requirement: Claude Code can be launched inside the sandbox via kit
The system SHALL provide a `cage claude` entry point that applies the `claude-kit` and starts Claude Code with `--dangerously-skip-permissions`.

#### Scenario: Standard launch
- **WHEN** the user runs `cage claude` from a project directory
- **THEN** `sbx run claude --kit <code-cage>/claude-kit -- --dangerously-skip-permissions` executes, the current directory is mounted as workspace, `~/.claude/` is available read-only, and `ANTHROPIC_API_KEY` is injected via the kit

#### Scenario: Skills and plugins are available
- **WHEN** Claude Code starts inside the sandbox
- **THEN** it loads skills (ponytail, superpowers, context7) from the kit-mounted `~/.claude/` exactly as it would on the host

