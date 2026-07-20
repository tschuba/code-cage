## ADDED Requirements

### Requirement: Cage session shell is visually distinct from host shell
Inside a cage session, the bash shell SHALL display a distinctive prompt prefix and set the terminal tab title to identify both the cage context and the agent name, so a user can immediately tell whether any given terminal is a cage session.

#### Scenario: Prompt shows cage indicator
- **WHEN** a user is at the bash prompt inside a cage session
- **THEN** the prompt prefix shows `⬡ cage:<agent>` (e.g., `⬡ cage:claude` or `⬡ cage:pi`) followed by the current working directory and `$`

#### Scenario: Terminal tab title shows cage indicator
- **WHEN** a cage session shell starts
- **THEN** the terminal emulator's tab/window title is set to `⬡ cage:<agent>` via an OSC escape sequence embedded in PS1

#### Scenario: Tab title persists while agent runs
- **WHEN** the agent (Claude Code or pi) is running and has taken over the terminal
- **THEN** the tab title remains `⬡ cage:<agent>` until the agent sets its own title or the session ends

#### Scenario: Indicator includes agent name
- **WHEN** the session was launched with `cage claude`
- **THEN** the indicator reads `⬡ cage:claude`

- **WHEN** the session was launched with `cage pi`
- **THEN** the indicator reads `⬡ cage:pi`

### Requirement: CAGE_AGENT env var is set inside the session
The cage container SHALL expose a `CAGE_AGENT` environment variable containing the agent name, so scripts and tools inside the session can detect and act on the cage context.

#### Scenario: Env var available after session start
- **WHEN** a user or script runs `echo $CAGE_AGENT` inside a cage session
- **THEN** the output is the agent name (`claude` or `pi`)

#### Scenario: Env var absent on host
- **WHEN** a user runs `echo $CAGE_AGENT` on the host terminal (outside cage)
- **THEN** the output is empty
