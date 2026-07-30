## Context

See proposal.md — Why. Claude Code stores conversation history under `~/.claude/projects/<hashed-path>/` relative to the agent's home. Inside the cage this is `/home/agent/.claude/projects/`, which is ephemeral. The host's `~/.claude/` is mounted `:ro`, so we cannot write there directly.

## Goals / Non-Goals

**Goals:**
- Conversation history written inside a cage session survives to the host
- `claude --resume` works in a subsequent cage for the same machine

**Non-Goals:**
- Automatic resume on cage startup (user opts in with `--resume`)
- pi.dev conversation persistence (pi conversations are browser/server-side; no local file store to persist)
- Per-project isolation of history (Claude Code already namespaces by project path internally)

## Decisions

**Separate host directory (`~/.cage/claude-projects/`) over rw re-mount of `~/.claude/`**

Making `~/.claude/` writable inside the cage would let the agent modify host Claude config (settings, credentials, installed plugins). A dedicated `~/.cage/claude-projects/` directory scopes the rw surface to conversation history only.

**Copy-in / background-sync over symlink**

sbx pre-creates or bind-mounts `/home/agent/.claude/projects/` as a directory that cannot be replaced by a symlink — `ln -sfn TARGET /home/agent/.claude/projects` falls back to creating a symlink named `claude-projects` inside the directory instead of at it.

The install command therefore uses a copy approach:
- On startup: `cp -ru $CAGE_PROJECTS/. /home/agent/.claude/projects/` seeds the session with prior history
- Background loop: `while true; do sleep 10; cp -ru /home/agent/.claude/projects/. $CAGE_PROJECTS/; done &` syncs new writes back to the host continuously

```sh
SRC=$(ls -d /Users/*/.claude 2>/dev/null | head -1)
CAGE_PROJECTS="$(dirname "$SRC")/.cage/claude-projects"
mkdir -p "$CAGE_PROJECTS" /home/agent/.claude/projects
cp -ru "$CAGE_PROJECTS/." /home/agent/.claude/projects/ 2>/dev/null || true
(while true; do sleep 10; cp -ru /home/agent/.claude/projects/. "$CAGE_PROJECTS/" 2>/dev/null || true; done) &
```

**`cage` script creates the dir before mounting**

`sbx run` requires mount sources to exist. `mkdir -p "$HOME/.cage/claude-projects"` runs before `sbx run`.

## Risks / Trade-offs

History accumulates indefinitely → Claude Code's own `--dangerously-delete-conversation-history` covers cleanup; no cage-specific solution needed.

Concurrent cage sessions write to the same projects dir → Claude Code uses per-project subdirs keyed by path hash; concurrent writes to different projects are safe. Concurrent writes to the same project from two cages are the same risk as running two Claude Code instances on the host, which is an existing edge case.

## Open Questions

None — pi.dev is explicitly out of scope; the symlink approach is fully determined.
