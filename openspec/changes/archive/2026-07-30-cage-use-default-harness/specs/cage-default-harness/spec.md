## Purpose

Allows users to configure a global default harness for cage so that running `cage` without arguments uses their preferred harness instead of always defaulting to `claude`.

## ADDED Requirements

### Requirement: User can set a persistent default harness
The `cage` script SHALL accept a `use <harness>` subcommand that writes the specified harness name to `~/.cage/config` as the persistent global default.

#### Scenario: Setting a default harness
- **WHEN** the user runs `cage use pi`
- **THEN** `~/.cage/config` is written with the default harness set to `pi`
- **AND** a confirmation message is printed

#### Scenario: Overwriting an existing default
- **WHEN** a default harness is already set in `~/.cage/config`
- **AND** the user runs `cage use claude`
- **THEN** the config is updated to `claude` and the old value is replaced

### Requirement: cage reads the default harness from config when no argument is given
When invoked without a harness argument, `cage` SHALL read `~/.cage/config` and use the configured default harness if present.

#### Scenario: Running cage with a configured default
- **WHEN** `~/.cage/config` sets the default harness to `pi`
- **AND** the user runs `cage` with no arguments
- **THEN** the `pi` harness is started (equivalent to `cage pi`)

#### Scenario: Running cage with no config file
- **WHEN** `~/.cage/config` does not exist or has no default set
- **AND** the user runs `cage` with no arguments
- **THEN** the `claude` harness is started (same behavior as before)

### Requirement: Explicit harness argument always takes precedence
If the user passes a harness name explicitly, it SHALL override any configured default.

#### Scenario: Explicit argument overrides config
- **WHEN** `~/.cage/config` sets the default harness to `pi`
- **AND** the user runs `cage claude`
- **THEN** the `claude` harness is started
