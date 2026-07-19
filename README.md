# code-cage

Secure sandbox launcher for AI coding agents. Runs Claude Code (and others) inside a Docker microVM so agents can operate with full permissions without touching host credentials or the home directory.

```sh
cage claude   # Claude Code with --dangerously-skip-permissions, isolated
cage pi       # pi coding agent, isolated
```

## How it works

`cage` wraps Docker [`sbx`](https://github.com/docker/sbx) — a microVM-based sandbox purpose-built for AI agents. Each invocation:

1. Removes any stale sandbox for the current directory
2. Starts a fresh microVM with only `$(pwd)` and `~/.claude/` (read-only) mounted
3. Passes `ANTHROPIC_API_KEY` from the host environment
4. Launches the agent with full permissions inside the VM

`~/.ssh`, `~/.aws`, and the rest of `$HOME` are never accessible.

## Requirements

- macOS / Apple Silicon
- [`sbx`](https://github.com/docker/sbx) CLI (no Docker Desktop required)
- A free Docker account
- [`pngpaste`](https://github.com/jcsalterego/pngpaste) for clipboard image sync (`brew install pngpaste`)

## Install

```sh
brew install docker/tap/sbx
sbx login
git clone https://github.com/tschuba/code-cage
cd code-cage && ./install
```

`./install` links `cage` and `cage-clipd` onto your PATH, installs `pngpaste`, and prints the WezTerm snippet for clipboard image paste if not yet configured.


## Usage

Run from within any project directory:

```sh
cage claude          # start Claude Code
cage pi              # start pi
cage claude --help   # pass args through to the agent
```

Each directory gets its own sandbox (`claude-<dirname>`). Starting a new session always wipes the previous one — sandboxes are ephemeral by design.

## Adding agents

Create `<agent>-kit/spec.yaml` in this repo. Use `kind: mixin` to extend a built-in agent (like `claude`), or `kind: agent` for a fully custom one (like `pi`). See `claude-kit/` and `pi-kit/` for examples.
