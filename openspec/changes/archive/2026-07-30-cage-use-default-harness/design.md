## Context

See proposal.md — Why. The `cage` script currently hardcodes `claude` as the default via `AGENT="${1:-claude}"`. The config file `~/.cage/` already exists (used for venv, logs, tmpdir, open requests), so a `config` file there is a natural home.

## Goals / Non-Goals

**Goals:**
- `cage use <harness>` persists the default to `~/.cage/config`
- `cage` with no arg reads that file and falls back to `claude`

**Non-Goals:**
- Per-project defaults
- Multiple named profiles
- Listing or inspecting the current default (just `cat ~/.cage/config`)

## Decisions

**Config format: `key=value` shell-style, one line**

```sh
default_harness=pi
```

`cage` is a POSIX sh script. A single-line key=value file can be sourced directly with `. ~/.cage/config` — no parser needed. Alternatives like JSON or TOML would require external tools.

**`cage use` as a subcommand, not a separate binary**

The `use` subcommand branches early in `cage` before the harness resolution logic, writes the file, and exits. No new files; the existing `cage` script is the only entry point users need to remember.

**Implementation sketch:**

```sh
# subcommand: cage use <harness>
if [ "$1" = "use" ]; then
    echo "default_harness=${2:?usage: cage use <harness>}" > "$HOME/.cage/config"
    echo "Default harness set to: $2"
    exit 0
fi

# read default if no arg given
if [ -z "$1" ] && [ -f "$HOME/.cage/config" ]; then
    . "$HOME/.cage/config"
fi
AGENT="${1:-${default_harness:-claude}}"
```

## Risks / Trade-offs

- `.` (source) executes the config file — if the file is tampered with it could run arbitrary code. Acceptable: `~/.cage/` is user-owned, same threat model as `.zshrc`.
- Dead harness names are not validated at `cage use` time — a typo won't surface until `cage` is run. Acceptable for now; the error from `sbx` is clear enough.
