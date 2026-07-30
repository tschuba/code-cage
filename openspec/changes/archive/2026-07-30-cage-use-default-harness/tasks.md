## 1. cage Script

- [x] 1.1 Add `cage use <harness>` subcommand: branch on `$1 = "use"`, write `default_harness=$2` to `~/.cage/config`, print confirmation, exit
- [x] 1.2 Read config on startup: source `~/.cage/config` if it exists and `$1` is empty
- [x] 1.3 Update `AGENT` assignment: change `"${1:-claude}"` → `"${1:-${default_harness:-claude}}"`

## 2. Verification

- [x] 2.1 Run `cage use pi` — confirm `~/.cage/config` contains `default_harness=pi`
- [x] 2.2 Run `cage` with no args — confirm `pi-kit` starts
- [x] 2.3 Run `cage claude` — confirm `claude-kit` starts despite config
- [x] 2.4 Remove `~/.cage/config` and run `cage` — confirm `claude` is the fallback
- [ ] 2.5 Run `openspec archive` to merge `cage-default-harness` spec into `openspec/specs/`
