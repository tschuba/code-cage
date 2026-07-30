## Requirements

### Requirement: `code .` launches VS Code on the host from inside the cage
The system SHALL provide a `code` command inside the sandbox that opens the specified path in VS Code on the host machine, via a host-side daemon and a shared request directory.

#### Scenario: Agent runs `code .`
- **WHEN** the agent runs `code .` inside the sandbox
- **THEN** VS Code opens on the host with the current working directory within 2 seconds

#### Scenario: Agent runs `code <file>`
- **WHEN** the agent runs `code path/to/file.ts` inside the sandbox
- **THEN** VS Code opens the specified file on the host

#### Scenario: cage-opend not running
- **WHEN** the agent runs `code .` and `cage-opend` is not running on the host
- **THEN** the command exits with an error message indicating the launch daemon is unavailable

### Requirement: cage-opend daemon starts with the cage
The `cage` launch script SHALL start the `cage-opend` daemon on the host if it is not already running, before launching the sandbox.

#### Scenario: Daemon auto-starts
- **WHEN** the user runs `cage claude` and `cage-opend` is not running
- **THEN** `cage-opend` is started in the background before the sandbox launches

#### Scenario: Daemon already running
- **WHEN** `cage-opend` is already running from a previous cage session
- **THEN** `cage` does not start a second instance

### Requirement: Launch requests are scoped and non-colliding
Request files written by the cage to `~/.cage/open/` SHALL include the cage name and a timestamp in the filename so concurrent cage sessions do not interfere with each other.

#### Scenario: Two simultaneous cage sessions both run `code .`
- **WHEN** two cage sessions each run `code .` at the same time
- **THEN** both VS Code windows open correctly without either request being lost or corrupted
