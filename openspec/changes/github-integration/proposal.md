## Why

Agents running inside cage sessions have no access to git remotes or the GitHub API — they can only commit locally. This makes autonomous workflows (push, PR creation, code review) impossible without manual intervention on the host.

## What Changes

- `cage` script extracts `GH_TOKEN` from the host via `gh auth token` and mounts `~/.gitconfig` read-only at launch time
- `claude-kit/spec.yaml` gains GitHub network allowlist, credential forwarding, proxy-level auth injection, and a git credential helper install command

## Capabilities

### New Capabilities

- `github-access`: Full git and GitHub API access inside cage sessions — commit, push, pull, `gh` CLI (PR creation, issue management, etc.) — authenticated via host's existing `gh` auth token, without mounting host credential directories

### Modified Capabilities

- `sbx-environment`: Token extraction and gitconfig mount added to the sandbox launch contract
- `agent-launch`: Launch sequence now conditionally injects `GH_TOKEN` and `~/.gitconfig` when available on the host

## Impact

- `cage` (shell script): 2–3 lines added before `sbx run`
- `claude-kit/spec.yaml`: new `credentials`, `network`, `environment`, and `commands.install` sections
- No new files, no new dependencies
- Backward compatible: if `gh auth token` fails (not logged in), cage falls back to launching without GitHub access
