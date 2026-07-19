## 1. Install & Authenticate sbx

- [x] 1.1 Run `brew trust docker/tap && brew install docker/tap/sbx` to install the sbx CLI
- [x] 1.2 Run `sbx login` and complete Docker OAuth authentication in the browser

## 2. Discover Kit spec.yaml Format

- [x] 2.1 Create a minimal `claude-kit/spec.yaml` skeleton and run `sbx kit validate claude-kit/` to see validation errors — use them to learn the required schema
- [x] 2.2 Search for Docker-published example kits (GitHub, Docker Hub) to find a working `spec.yaml` reference

## 3. Create claude-kit

- [x] 3.1 Write `claude-kit/spec.yaml` with a BindMount for `~/.claude/` read-only at the same absolute path
- [x] 3.2 Add `ANTHROPIC_API_KEY` as a credential in the kit spec
- [x] 3.3 Add context7 and other active plugins as `static-mcp` entries (if needed — plugins may auto-load via the mounted `~/.claude/`)
- [x] 3.4 Run `sbx kit validate claude-kit/` and fix any errors

## 4. Create cage Wrapper Script

- [x] 4.1 Create `cage` shell script that maps `cage <agent>` to `sbx run <agent> --kit <code-cage>/<agent>-kit -- <agent-args>`
- [x] 4.2 Make `cage` executable and add to `$PATH` (symlink in `/usr/local/bin` or shell alias in `.zshrc`)

## 5. Verification

- [x] 5.1 Run `cage claude` from a test project and confirm Claude starts with skills and plugins available (check ponytail, context7)
- [x] 5.2 Confirm `~/.ssh` and `~/.aws` are not accessible from inside the sandbox
- [ ] 5.3 Confirm `cage pi` launches pi in a sandboxed shell
