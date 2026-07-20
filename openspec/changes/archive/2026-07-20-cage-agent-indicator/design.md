## Context

Each cage session runs inside an sbx microVM. The kit's `install:` commands execute during container setup as bash commands with configurable users. There is currently no visual distinction between a cage shell and the host shell — both show a plain bash prompt.

`sbx run` has no `-e` flag for injecting env vars at launch time. The only injection point available without changing the cage script is the kit's `install:` commands, which can write to `/home/agent/.bashrc`.

## Goals / Non-Goals

**Goals:**
- Cage sessions show a distinctive shell prompt prefix and terminal tab title
- The indicator names the agent (`claude` or `pi`)
- Works at the shell prompt AND while the agent is running (tab title persists)
- Zero host-side changes

**Non-Goals:**
- Customizing the indicator's appearance beyond a simple prefix
- Syncing state back to the host terminal
- Supporting agents other than claude and pi

## Decisions

**Write to `/home/agent/.bashrc` via install command**

The kit install step is the only container-side hook available. Writing two lines to `.bashrc` sets `CAGE_AGENT` and prepends the prompt/title escape. Alternative (writing to `/etc/environment`) only sets the var, not the prompt; alternative (modifying `cage` script) would require host-side changes and sbx doesn't pass `-e` env flags anyway.

**Terminal title via PS1 OSC escape**

Embedding `\[\e]0;⬡ cage:$CAGE_AGENT\a\]` in PS1 sets the tab title on every prompt redraw. This means the title is set before the agent launches (first prompt) and restored when the agent exits (next prompt). While the agent runs, the title sticks at whatever it was last set — which is `⬡ cage:claude` (or pi), giving visibility even with the agent's TUI in the foreground.

Claude Code may emit its own title escapes; the cage title would then be overridden while Claude Code is active, but restored on exit. Acceptable trade-off — the host tab title is the secondary signal; the prompt covers the at-shell case.

**Same indicator format for both kits**

`⬡ cage:claude` / `⬡ cage:pi` — consistent, scannable. The `⬡` glyph is distinctive without requiring a Nerd Font.

## Risks / Trade-offs

- Agent overrides terminal title while running → title restores on exit; prompt prefix always present at shell
- `.bashrc` is sourced only for interactive shells; if the agent is launched non-interactively this won't apply → not an issue, cage sessions are always interactive
- User's host `.bashrc` is not mounted into the container, so no conflict with host prompt config
