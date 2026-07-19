## ADDED Requirements

### Requirement: Only the current project directory is mounted
The sandbox SHALL mount exclusively the directory from which the launch command is invoked, at the same absolute path as on the host. No other host directories SHALL be accessible inside the sandbox.

#### Scenario: Launch from project directory
- **WHEN** the user runs `cage claude` from `/Users/thomas/Projects/myapp`
- **THEN** `/Users/thomas/Projects/myapp` is available inside the sandbox at the same path, and no parent or sibling directories are accessible

#### Scenario: Credentials stay on host
- **WHEN** the agent runs inside the sandbox and attempts to read `~/.ssh/id_rsa` or `~/.aws/credentials`
- **THEN** the read fails — those paths do not exist inside the sandbox

### Requirement: API keys are passed as environment variables only
Agent credentials (API keys) SHALL be injected as environment variables at sandbox launch time. No credential files SHALL be mounted into the sandbox.

#### Scenario: Claude Code API key
- **WHEN** `ANTHROPIC_API_KEY` is set in the host shell environment
- **THEN** it is forwarded into the sandbox as an environment variable and Claude Code can authenticate without any file-based credential

#### Scenario: Missing API key
- **WHEN** the required API key environment variable is not set on the host
- **THEN** the launch script exits with a clear error message before starting the sandbox
