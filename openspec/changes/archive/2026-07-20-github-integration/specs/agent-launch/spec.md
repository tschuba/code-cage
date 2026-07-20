## MODIFIED Requirements

### Requirement: Claude Code can be launched inside the sandbox via kit
The system SHALL provide a `cage claude` entry point that applies the `claude-kit` and starts Claude Code with `--dangerously-skip-permissions`. When the host has a GitHub token stored in the sbx secret store, the agent SHALL have full git and GitHub API access.

#### Scenario: Standard launch
- **WHEN** the user runs `cage claude` from a project directory
- **THEN** `sbx run claude --kit <code-cage>/claude-kit -- --dangerously-skip-permissions` executes, the current directory is mounted as workspace, `~/.claude/` is available read-only, and `ANTHROPIC_API_KEY` is injected via the kit

#### Scenario: Skills and plugins are available
- **WHEN** Claude Code starts inside the sandbox
- **THEN** it loads skills (ponytail, superpowers, context7) from the kit-mounted `~/.claude/` exactly as it would on the host

#### Scenario: GitHub access available when sbx secret is configured
- **WHEN** the host has run `sbx secret set -g github` with a valid GitHub token
- **THEN** `GH_TOKEN` is set inside the sandbox (via sbx proxy mechanism), `gh` CLI is authenticated, and git HTTPS operations succeed against GitHub

#### Scenario: Launch succeeds without GitHub secret
- **WHEN** no GitHub token is stored in the sbx secret store
- **THEN** cage launches the sandbox normally; `GH_TOKEN` is not set; git remote and `gh` CLI operations fail with clear authentication errors, but local git works normally
