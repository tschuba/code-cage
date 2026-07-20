## ADDED Requirements

### Requirement: Agent can perform git remote operations against GitHub
Inside a cage session, the agent SHALL be able to clone, fetch, pull, and push to GitHub repositories over HTTPS using the host user's GitHub credentials.

#### Scenario: git push succeeds
- **WHEN** the agent runs `git push` inside the sandbox and a GitHub token is stored in the sbx secret store
- **THEN** the push completes successfully against the GitHub remote via the sbx proxy mechanism, without prompting for credentials

#### Scenario: git clone of private repo succeeds
- **WHEN** the agent runs `git clone https://github.com/<owner>/<repo>` for a repo the host user has access to
- **THEN** the clone completes successfully inside the sandbox workspace

#### Scenario: No host credential files exposed
- **WHEN** git remote operations succeed inside the sandbox
- **THEN** `~/.ssh/`, `~/.config/gh/`, and `~/.netrc` are NOT accessible inside the sandbox — auth is handled by the sbx proxy using a stored secret, not by credential files

### Requirement: Agent can use gh CLI for GitHub API operations
The `gh` CLI SHALL be available and authenticated inside the cage session, enabling PR creation, issue management, and GitHub API access.

#### Scenario: gh pr create succeeds
- **WHEN** the agent runs `gh pr create` inside the sandbox and a GitHub token is stored via `sbx secret set -g github`
- **THEN** the PR is created on GitHub without any interactive auth prompt

#### Scenario: gh auth status shows authenticated
- **WHEN** a GitHub token is stored in the sbx secret store
- **THEN** `gh auth status` inside the sandbox reports an authenticated session (via `GH_TOKEN` set by the sbx proxy mechanism)

### Requirement: Agent commits carry correct git identity
Commits made inside the sandbox SHALL use the project's configured git identity.

#### Scenario: Commit has correct author
- **WHEN** the agent makes a git commit inside the sandbox on the mounted workspace
- **THEN** the commit's author name and email are read from the project's `.git/config` (repo-level identity, inherited from the host-mounted workspace)

### Requirement: GitHub access degrades gracefully when no token is stored
If no GitHub token is stored in the sbx secret store, cage SHALL still launch successfully with local-only git capabilities.

#### Scenario: No sbx github secret configured
- **WHEN** `sbx secret set -g github` has not been run on the host
- **THEN** cage launches the sandbox normally; git remote and `gh` CLI operations fail with clear authentication errors, but local git and sandbox functionality work normally
