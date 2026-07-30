# cage-parallel-sessions Specification

## Purpose
Allows multiple cage sessions to run concurrently — for the same project in separate terminals, or for different projects that share a directory basename — without sessions interfering with each other.
## Requirements
### Requirement: Each cage invocation gets a unique session identity
Each `cage` invocation SHALL be assigned a session name that is unique to that process, so that starting a new cage session never terminates an existing one.

#### Scenario: Second terminal does not kill first
- **WHEN** a cage session is already running for project `~/work/myapp`
- **AND** the user opens a second terminal and runs `cage` in the same directory
- **THEN** the first session continues running unaffected

#### Scenario: Same-basename projects do not collide
- **WHEN** cage is running in `~/work/myapp`
- **AND** the user starts cage in `~/personal/myapp`
- **THEN** both sessions run concurrently with independent sandbox identities

### Requirement: A cage session never forcibly terminates a sibling session on startup
The `cage` script SHALL NOT remove or stop any existing sandbox at startup.

#### Scenario: Pre-existing session survives new cage launch
- **WHEN** an sbx sandbox named `claude-myapp-<pid1>` is running
- **AND** the user runs `cage` again in the same directory, producing `claude-myapp-<pid2>`
- **THEN** `claude-myapp-<pid1>` continues running

#### Scenario: Dead sessions are not cleaned up automatically
- **WHEN** a previous cage session ended and its sandbox no longer exists
- **AND** the user runs `cage` in the same directory
- **THEN** a new session starts without error (no attempt to rm a nonexistent sandbox)

