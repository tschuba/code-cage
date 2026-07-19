## ADDED Requirements

### Requirement: Agent can perform git remote operations against GitHub
Inside a cage session, the agent SHALL be able to clone, fetch, pull, and push to GitHub repositories over HTTPS using the host user's GitHub credentials.

#### Scenario: git push succeeds
- **WHEN** the agent runs `git push` inside the sandbox
- **THEN** the push completes successfully against the GitHub remote using the injected token, without prompting for credentials

#### Scenario: git clone of private repo succeeds
- **WHEN** the agent runs `git clone https://github.com/<owner>/<repo>` for a repo the host user has access to
- **THEN** the clone completes successfully inside the sandbox workspace

#### Scenario: No host credential files exposed
- **WHEN** git remote operations succeed inside the sandbox
- **THEN** `~/.ssh/`, `~/.config/gh/`, and `~/.netrc` are NOT accessible inside the sandbox — auth is via injected token only

### Requirement: Agent can use gh CLI for GitHub API operations
The `gh` CLI SHALL be available and authenticated inside the cage session, enabling PR creation, issue management, and GitHub API access.

#### Scenario: gh pr create succeeds
- **WHEN** the agent runs `gh pr create` inside the sandbox
- **THEN** the PR is created on GitHub using the injected token without any interactive auth prompt

#### Scenario: gh auth status shows authenticated
- **WHEN** the agent runs `gh auth status` inside the sandbox
- **THEN** it reports an authenticated session (via `GH_TOKEN` env var)

### Requirement: Agent commits carry correct git identity
Commits made inside the sandbox SHALL use the host user's git identity (name and email).

#### Scenario: Commit has correct author
- **WHEN** the agent makes a git commit inside the sandbox
- **THEN** the commit's author name and email match the host user's `git config user.name` and `git config user.email`

### Requirement: GitHub access degrades gracefully when host has no token
If the host has no active GitHub authentication, cage SHALL still launch successfully with local-only git capabilities.

#### Scenario: No gh auth on host
- **WHEN** `gh auth token` returns empty or fails on the host
- **THEN** cage launches the sandbox without `GH_TOKEN` and without the gitconfig mount; git remote operations will fail inside the sandbox but local git works normally
