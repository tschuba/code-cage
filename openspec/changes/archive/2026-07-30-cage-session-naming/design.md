## Context

See proposal.md — Why. The `cage` script currently builds a deterministic name (`${AGENT}-${basename(pwd)}`) and immediately runs `sbx rm -f "$NAME"` before starting, which destroys any session sharing that name.

## Goals / Non-Goals

**Goals:**
- Each cage invocation gets a name unique to its process
- No existing session is ever killed on startup

**Non-Goals:**
- Session listing or reattach (`sbx attach`)
- Automatic cleanup of orphaned sandboxes
- Changing tab title or prompt identity (those use `$AGENT`, not `$NAME`)

## Decisions

**PID suffix over timestamp**

`$$` (shell PID) is guaranteed unique for the lifetime of the process, requires no external command, and appears naturally in `pgrep` output for debugging. A timestamp risks collision if two cages start in the same second; a random suffix is harder to read.

```sh
# before
NAME="${AGENT}-$(basename "$(pwd)" | tr -cs 'a-zA-Z0-9.' '-' | sed 's/-$//')"
sbx rm -f "$NAME" 2>/dev/null || true

# after
NAME="${AGENT}-$(basename "$(pwd)" | tr -cs 'a-zA-Z0-9.' '-' | sed 's/-$//')-$$"
```

The `sbx rm -f` line is deleted entirely.

## Risks / Trade-offs

Dead sandbox accumulation → not a problem in practice: `sbx` sandboxes are lightweight and die with the process. If manual cleanup is ever needed, `sbx ls` + `sbx rm` work as before.

`cage-opend` writes request files named `${n}-${t}.json` where `n` comes from `SANDBOX_VM_ID` or `hostname` inside the cage — unaffected by this change.
