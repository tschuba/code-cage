## Purpose

Preserves Claude Code conversation history across cage sessions so users can resume a prior conversation with `claude --resume` in a new cage session for the same machine.

## ADDED Requirements

### Requirement: Conversation history persists after a cage session ends
Claude Code conversation history written inside a cage session SHALL survive sandbox teardown and be readable in a subsequent cage session on the same host.

#### Scenario: History available after closing cage
- **WHEN** the user runs `cage`, has a conversation with Claude Code, and then exits the cage session
- **THEN** the conversation history files exist on the host at `~/.cage/claude-projects/`

#### Scenario: Resume works in a new cage session
- **WHEN** the user starts a new cage session and runs `claude --resume`
- **THEN** Claude Code resumes the most recent prior conversation from that project

### Requirement: Each cage session starts a fresh conversation by default
Starting `cage` without flags SHALL launch Claude Code without any `--resume` or `--continue` flag, so the default experience is a clean session.

#### Scenario: Default cage launch is fresh
- **WHEN** the user runs `cage` without additional arguments
- **THEN** Claude Code starts with no active conversation (new session)

#### Scenario: Resume is opt-in
- **WHEN** the user wants to resume a prior conversation
- **THEN** they pass `--resume` (or the equivalent flag) directly to claude inside the cage, or as an argument to `cage`

### Requirement: Conversation history is shared across all cage sessions on the same host
The persistent projects directory SHALL be a single host-level store (not per-project), matching Claude Code's own project-based organisation of conversation history.

#### Scenario: History from one cage session is visible in another
- **WHEN** the user has run cage in `~/work/projectA` and `~/work/projectB`
- **THEN** `claude --resume` inside either cage session can navigate history from both projects
