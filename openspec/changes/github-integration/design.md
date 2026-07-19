## Context

cage agents can commit locally but have no access to git remotes or the GitHub API. The host's GitHub token lives in the macOS Keychain — it's not a file that can be mounted. `gh auth token` is the only reliable way to extract it at runtime.

The sbx credential model uses `credentials.sources` (reads from host env) + `proxyManaged` (makes the credential available inside the sandbox as an env var and uses it for proxy-level service auth). This is the same mechanism ANTHROPIC_API_KEY already uses.

## Goals / Non-Goals

**Goals:**
- Full git remote operations (clone, push, pull) via HTTPS inside the sandbox
- `gh` CLI available and authenticated for PR/issue/API workflows
- Correct git identity (user.name / user.email) on commits
- Graceful degradation: launch succeeds even if host has no GitHub auth

**Non-Goals:**
- SSH-based git (host uses HTTPS; adding SSH key management adds complexity for no gain)
- Fine-grained token scoping per-repo (out of scope; use a fine-grained PAT if needed separately)
- Persisting gh state written inside the sandbox (ephemeral by design)

## Decisions

### D1: Extract token via `gh auth token`, not by mounting `~/.config/gh/`

`gh auth status` shows the token is stored in the macOS Keychain (`(keyring)`). Mounting `~/.config/gh/` would give the sandbox the config file but not the keychain — `gh` inside the sandbox would fail to authenticate. `gh auth token` prints the raw token regardless of storage backend.

**Alternative considered:** mount `~/.config/gh/:ro` — rejected because keychain access is not available inside the sbx microVM.

### D2: Inject token via `credentials.sources` + `proxyManaged`, not `environment.variables`

`environment.variables` requires a static value in spec.yaml — hardcoding a token there is a security anti-pattern and would end up in git history. The `credentials.sources.env` mechanism reads the value from the host environment at launch time; `proxyManaged` makes it available inside the sandbox as a real env var (for `gh` CLI) and configures the outbound proxy to inject `Authorization: Bearer <token>` for github.com traffic (for HTTPS git).

**Alternative considered:** pass token as a plain env var via the cage script's shell environment — this works but bypasses sbx's credential model and doesn't get proxy-level auth injection.

### D3: Proxy-level auth for git HTTPS, with credential helper as fallback

The sbx proxy injects the Bearer token for all traffic to `serviceDomains`. For most `gh` CLI and GitHub API calls this is sufficient. Git HTTPS sends its own `Authorization` header via the credential helper, which may or may not pass through the proxy. An `install` command configures a minimal credential helper (`!f() { echo username=x-access-token; echo password=$GH_TOKEN; }; f'`) as a fallback so git push/pull work even if the proxy doesn't intercept credential exchange.

### D4: Git identity via `~/.gitconfig:ro` mount

Mounting `~/.gitconfig` read-only at the same absolute path propagates the full host git identity (user.name, user.email, default branch, aliases, etc.) without any additional env var plumbing. sbx mounts at the same path as on the host, which works because the agent user home inside the microVM mirrors the host path.

### D5: Graceful degradation if no GitHub auth on host

`GH_TOKEN=$(gh auth token 2>/dev/null)` produces an empty string if `gh` is not installed or not authenticated. The cage script SHALL skip token injection if the value is empty. The sandbox launches normally with local-only git capabilities.

## Risks / Trade-offs

- **Token scope is broad** → The host `gh` token has `repo` + `workflow` scopes. An agent can push to any repo the user has access to. Mitigation: use a fine-grained PAT with repo-specific permissions if stricter control is needed (not in scope for this change).
- **Token leaks on host shell history** → `GH_TOKEN` is exported as an env var in the cage script; it's not written to any file. Risk is low.
- **gitconfig side effects** → Agent inherits all host git config including aliases and hooks. Unlikely to cause issues; document if it becomes one.
