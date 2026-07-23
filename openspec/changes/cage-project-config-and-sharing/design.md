## Context

`cage` is a shell script that wraps `sbx run`. It currently passes three fixed mounts (project dir, `~/.claude:ro`, `~/.cage/clipboard:ro`) and relies on the kit's `spec.yaml` for network domain allowlists. There is no mechanism for projects or users to declare additional mounts or domains without editing source files. `~/.cage/clipboard/` exists as a runtime sharing channel but is clipboard-daemon-managed and global — not project-scoped.

## Goals / Non-Goals

**Goals:**
- Three-level config hierarchy: global → project (committed) → project-local (gitignored)
- `mounts:` declarable in global and project-local configs; network `allowedDomains:` declarable in all three levels
- `cage` auto-creates `PROJECT/.cage/` and `PROJECT/.cage/.gitignore` on first launch
- A `cage-config` Python helper handles YAML parsing and config merging
- `PROJECT/.cage/share/` and `PROJECT/.cage/scratch/` established as named conventions with agent-facing documentation
- Clipboard bridge (`~/.cage/clipboard/`) left entirely unchanged

**Non-Goals:**
- Hot-mounting directories into a running sandbox (sbx does not support this)
- Relative mount paths in project config (paths are machine-specific; relative mounts would require a consistent checkout layout assumption)
- Restricting or overriding global/kit domains at project level (configs are additive only)
- Runtime mount requests from inside the sandbox

## Decisions

### Mounts are never in the committed project config

Mount paths contain `~/…` which resolve differently per machine. A teammate checking out the repo would inherit broken paths. Domains are genuinely machine-invariant project facts (`api.company-internal.com` is the same for every contributor), so they belong in the committed config. Mounts belong in `~/.cage/config.yaml` (global, personal) or `PROJECT/.cage/config.local.yaml` (project-scoped, personal, gitignored).

### Single Python helper (`cage-config`) rather than inline shell parsing

YAML has enough edge cases (comments, multi-line values, missing keys) that shell `grep`/`sed` parsing is fragile. Python 3 is already a declared dependency (cage-clipd uses it). `cage-config` is a small standalone script: no new deps, consistent with the existing toolset.

### Network domain injection: spike required

The kit `spec.yaml` is the current home for `network.allowedDomains`. There are two paths to inject additional domains at launch time:

**Path A — `sbx run` CLI flag** (preferred if available): `sbx run --allow-domain foo.com …`. Zero file generation, clean, no temp dir cleanup.

**Path B — temp merged kit**: cage generates a merged `spec.yaml` in a temp dir (`mktemp -d`), symlinking all other kit files, with additional domains appended. Passes `--kit /tmp/cage-merged-XXXX` to sbx. Requires cleanup on exit (cage drops `exec` and uses `sbx run; rm -rf "$TMPKIT"` instead).

The first task is a spike to determine which path sbx supports. The rest of the network domain implementation follows the spike outcome.

### `share/` and `scratch/` are convention, not mechanism

Both directories sit inside the project dir, which is already mounted read-write. No new sbx mounts are needed. The value is naming (CLAUDE.md tells the agent what each directory is for) and gitignore hygiene (auto-created `.cage/.gitignore` ensures they don't pollute the repo).

`share/` is bidirectional: user drops reference files there; agent writes outputs there. A single directory is simpler than separate `relay/` and `output/` dirs, since the mount enforces no direction and a two-bucket model would just create confusion about which to use.

`scratch/` is agent-internal: persistent notes, prior-session context (`CONTEXT.md`), research. The agent reads it at session start and writes to it throughout.

### Clipboard bridge stays separate

`~/.cage/clipboard/` is reactive (daemon-driven), transient (cleared on next clipboard event), and global (same content in all active cage sessions). `share/` is deliberate, persistent, and project-scoped. Merging them would require routing clipboard events per-project (a session registry, broadcast writes, cleanup logic) for no practical gain — the clipboard bridge already works well as-is.

## Config Schema

Same shape at all three levels; unrecognised keys are ignored:

```yaml
mounts:
  - ~/path/to/dir:ro      # :ro default, :rw also accepted
  - ~/other/path:ro

network:
  allowedDomains:
    - api.company-internal.com
    - staging.myapp.com
```

## Merge Logic

```
final_mounts   = global.mounts + local.mounts          (deduplicated, order preserved)
final_domains  = kit.domains + global.domains
               + project.domains + local.domains        (deduplicated, order preserved)
```

Project `config.yaml` contributes only domains. Global and local contribute only mounts (domains also accepted in both for flexibility, but the primary use is as above).

Missing config files are silently skipped. Empty sections are fine.

## `cage` Script Changes

```sh
# After deriving NAME, before exec:
CAGE_CONFIG="$CAGE_DIR/cage-config"

# Auto-create .cage/ workspace
python3 "$CAGE_CONFIG" --init "$(pwd)"

# Collect extra mounts from global + local configs
EXTRA_MOUNTS=$(python3 "$CAGE_CONFIG" --mounts \
    "$HOME/.cage/config.yaml" \
    "$(pwd)/.cage/config.local.yaml" 2>/dev/null)

# Network domains (spike determines how these are consumed)
EXTRA_DOMAINS=$(python3 "$CAGE_CONFIG" --domains \
    "$HOME/.cage/config.yaml" \
    "$(pwd)/.cage/config.yaml" \
    "$(pwd)/.cage/config.local.yaml" 2>/dev/null)

exec sbx run "$AGENT" --kit "$KIT" --name "$NAME" \
    . "$HOME/.claude:ro" "$HOME/.cage/clipboard:ro" \
    $EXTRA_MOUNTS \
    -- [flags] "$@"
```

## `.cage/.gitignore` (auto-created)

```
# Generated by cage — do not edit
config.local.yaml
scratch/
share/
```

`config.yaml` is not ignored — it is committed as project configuration.

## `cage-config` Interface

```
cage-config --init <project-dir>
    Create .cage/ and .cage/.gitignore if absent.

cage-config --mounts <config-file> [<config-file> ...]
    Print one mount per line (path:mode), deduplicated.

cage-config --domains <config-file> [<config-file> ...]
    Print one domain per line, deduplicated.

cage-config --merge-kit <kit-dir> <config-file> [...]
    Write a merged spec.yaml to stdout (Path B only, post-spike).
```

Missing files are silently skipped. Invalid YAML prints a warning to stderr and is skipped.

## Risks / Trade-offs

- **Network domain injection is unverified** — the sbx CLI may not support per-invocation domain flags; Path B (temp kit) adds complexity and requires dropping `exec` from the cage script. The spike resolves this before any implementation.
- **mount path resolution** — `~` in config files is expanded by `cage-config` using `os.path.expanduser`; this is the user who runs `cage`, which is always correct.
- **YAML as a shell-script dependency** — Python's `yaml` module is used; it ships with macOS system Python (3.x). No pip install required.
- **`.cage/` name collision** — unlikely but possible if a project already uses `.cage/` for something else. No mitigation; document the convention and move on.
