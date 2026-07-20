## 1. claude-kit

- [x] 1.1 Add install command to `claude-kit/spec.yaml` that appends `export CAGE_AGENT=claude` and cage-prefixed PS1 to `/home/agent/.bashrc`

## 2. pi-kit

- [x] 2.1 Add install command to `pi-kit/spec.yaml` that appends `export CAGE_AGENT=pi` and cage-prefixed PS1 to `/home/agent/.bashrc`

## 3. Verify

- [x] 3.1 Run `cage claude` in a test directory and confirm the shell prompt shows `⬡ cage:claude` and the terminal tab title updates
- [x] 3.2 Run `cage pi` and confirm `⬡ cage:pi` appears in prompt and tab title
- [x] 3.3 Confirm host terminal shows no indicator (`echo $CAGE_AGENT` is empty)
