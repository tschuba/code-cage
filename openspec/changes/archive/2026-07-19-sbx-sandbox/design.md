## Context

AI coding agents (Claude Code, pi, etc.) require broad system access to be useful — reading/writing files, running shell commands, installing packages. Running them with unrestricted permissions on a developer's host machine exposes credentials (`~/.ssh`, `~/.aws`), personal files, and system configuration to an autonomous process.

Docker's `sbx` CLI provides microVM-based isolation specifically designed for AI coding agents. Each sandbox gets its own Docker daemon, filesystem, and network. The host is not touched beyond the explicitly mounted workspace directory.

The user is on macOS 26 / Apple Silicon. No Docker Desktop is required.

## Goals / Non-Goals

**Goals:**
- Isolate AI agent execution so only the mounted project directory is accessible
- Keep host credentials and home directory completely out of reach
- Simple launch UX: one command per agent
- Extensible: adding a new agent is adding one script

**Non-Goals:**
- CI/CD or team/cloud use (sbx governance subscriptions, remote runners)
- Windows or Linux support (macOS-first)
- Persistent sandbox state between sessions — ephemeral is intentional
- Blocking outbound network (agents need API access)
- GUI or daemon — shell scripts only

## Decisions

### sbx over Apple `container run`

**Decision**: Use Docker `sbx` CLI as the sandbox runtime.

**Rationale**: `sbx` is purpose-built for AI coding agents. It sets up workspace passthrough, network proxying, and daemon isolation automatically. Apple `container run` requires manually building an OCI image with the agent installed and configuring mounts — more setup with no meaningful isolation advantage.

**Alternative considered**: Apple `container run` — rejected due to setup overhead; `container machine` rejected outright (auto-mounts `$HOME`, opposite of what we want).

### Ephemeral sandboxes

**Decision**: No persistent sandbox state. Each `sbx run` starts clean.

**Rationale**: Persistence accumulates state that's invisible to the user and hard to audit. Ephemeral is safer and simpler — installed packages inside the sandbox don't survive between sessions, but the mounted workspace does (it's the host filesystem).

### Workspace mount strategy

**Decision**: Mount only `$(pwd)` (the current project directory) into the sandbox.

**Rationale**: `sbx` passes the workspace at the same absolute path as on the host. Launching from within the project directory gives the agent exactly the right scope — no more, no less.

### Kit-based configuration

**Decision**: Use an sbx Kit (`claude-kit/spec.yaml`) as the primary configuration mechanism instead of passing flags manually.

**Rationale**: `sbx run claude --kit ./claude-kit` is a single reusable artifact that declaratively captures everything the sandbox needs: BindMount of `~/.claude/` read-only, API key credentials, and MCP server definitions via `static-mcp`. Without a kit, every invocation requires multiple flags and environment variable plumbing in a wrapper script. The kit moves that complexity into versioned config that lives in this repo.

**Structure**:
```
code-cage/
└── claude-kit/
    ├── spec.yaml     ← BindMount ~/.claude/:ro + credentials + static-mcp
    └── files/        ← optional static files injected into $HOME
```

**Launch**:
```bash
sbx run claude --kit /path/to/code-cage/claude-kit -- --dangerously-skip-permissions
```

### `~/.claude/` access via BindMount

**Decision**: Mount `~/.claude/` read-only into the sandbox via the kit's BindMount mechanism, not as a static file copy.

**Rationale**: A live mount means skills, plugins, and CLAUDE.md are always current without any sync step. `~/.claude/` contains no credentials (only config, skills, markdown), so read-only exposure is safe. The deny-list in `settings.json` (`~/.ssh`, `~/.aws`, etc.) remains enforced by the sandbox boundary itself.

### Credential passing

**Decision**: Declare `ANTHROPIC_API_KEY` (and other agent-specific keys) as credentials in `spec.yaml`. No credential files mounted.

**Rationale**: Kit credentials are the idiomatic sbx mechanism — scoped to the sandbox lifetime, sourced from the host environment at launch, not written to any filesystem inside the VM.

## Risks / Trade-offs

- **sbx requires a Docker account** (`sbx login`): one-time OAuth, free tier sufficient → acceptable friction
- **sbx is a relatively new tool**: fewer community resources than plain `docker run` → mitigated by Docker's official backing and the purpose-fit design
- **Kit `spec.yaml` format is experimental**: no public schema docs; format must be discovered via `sbx kit validate` iteration or community examples → treat kit as the first thing to spike before implementing anything else
- **Outbound network is open**: agents can exfiltrate data via the network — inherent to the use case (API calls needed) → document this clearly, not a fixable constraint without breaking agent functionality
- **macOS-only**: `sbx` for macOS only for now → noted as non-goal, acceptable
