## ADDED Requirements

### Requirement: Common home subdirectories are mounted read-only by default
The sandbox SHALL mount `~/Downloads`, `~/Documents`, and `~/Desktop` from the host as read-only paths inside the sandbox at the same absolute paths, so Claude can reference files the user has placed there without gaining write access. Credential directories (`~/.aws`, `~/.ssh`, `~/.gnupg`, `~/.netrc`) are excluded by not being mounted — they do not exist inside the sandbox.

#### Scenario: Claude reads a file from Downloads
- **WHEN** a file exists at `~/Downloads/report.pdf` on the host
- **THEN** the agent inside the sandbox can read it at `~/Downloads/report.pdf`

#### Scenario: Claude reads a file from Documents
- **WHEN** a file exists at `~/Documents/notes.md` on the host
- **THEN** the agent inside the sandbox can read it at `~/Documents/notes.md`

#### Scenario: Claude cannot write to the read-only mounts
- **WHEN** the agent attempts to write or delete a file under `~/Downloads`, `~/Documents`, or `~/Desktop`
- **THEN** the operation fails with a permission error

### Requirement: Credential paths are never mounted
The sandbox SHALL NOT mount `~/.aws`, `~/.ssh`, `~/.gnupg`, `~/.netrc`, or any path that is a known credential store, even when broader read access is configured.

#### Scenario: Agent attempts to read SSH key
- **WHEN** the agent running inside the sandbox tries to read `~/.ssh/id_rsa`
- **THEN** the read fails — the path does not exist inside the sandbox

#### Scenario: Agent attempts to read AWS credentials
- **WHEN** the agent attempts to read `~/.aws/credentials`
- **THEN** the read fails — the path does not exist inside the sandbox
