## ADDED Requirements

### Requirement: Kit install writes CAGE_AGENT and prompt config to .bashrc
Each cage kit's install commands SHALL append environment variable and PS1 configuration to `/home/agent/.bashrc` so that every interactive shell in the container inherits the cage session identity.

#### Scenario: claude-kit sets CAGE_AGENT=claude
- **WHEN** the claude-kit install commands run during container setup
- **THEN** `/home/agent/.bashrc` contains `export CAGE_AGENT=claude`

#### Scenario: pi-kit sets CAGE_AGENT=pi
- **WHEN** the pi-kit install commands run during container setup
- **THEN** `/home/agent/.bashrc` contains `export CAGE_AGENT=pi`

#### Scenario: PS1 is updated with cage indicator and terminal title escape
- **WHEN** a kit's install command runs
- **THEN** `/home/agent/.bashrc` contains a PS1 export that prepends `\[\e]0;⬡ cage:$CAGE_AGENT\a\]⬡ cage:$CAGE_AGENT ` to the existing prompt, setting both the tab title and visible prefix
