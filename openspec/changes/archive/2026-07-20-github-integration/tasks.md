## 1. cage script — token extraction and gitconfig mount

- [x] 1.1 Before `sbx run`, extract `GH_TOKEN=$(gh auth token 2>/dev/null)` and export it
- [x] 1.2 Add conditional gitconfig mount: if `GH_TOKEN` is non-empty and `~/.gitconfig` exists, add `"$HOME/.gitconfig:ro"` to the `sbx run` mount arguments

## 2. claude-kit/spec.yaml — GitHub network and credentials

- [x] 2.1 Add `network.allowedDomains` with `github.com`, `api.github.com`, `objects.githubusercontent.com`, `codeload.github.com`, `raw.githubusercontent.com`
- [x] 2.2 Add `network.serviceDomains` mapping `github.com` and `api.github.com` to service id `github`
- [x] 2.3 Add `network.serviceAuth.github` with `headerName: Authorization` and `valueFormat: "Bearer %s"`
- [x] 2.4 Add `credentials.sources.github.env: [GH_TOKEN]`
- [x] 2.5 Add `environment.proxyManaged: [GH_TOKEN]`

## 3. claude-kit/spec.yaml — git credential helper

- [x] 3.1 Add `commands.install` entry that runs as user `1000`: configure `git config --global credential.helper` to echo `username=x-access-token` and `password=$GH_TOKEN`

## 4. Smoke test

- [x] 4.1 Run `cage claude` from a test repo and verify `gh auth status` reports authenticated inside the sandbox
- [x] 4.2 Verify `git push` succeeds to a GitHub remote from inside the sandbox
- [x] 4.3 Verify `git log --format="%an <%ae>"` shows the correct host identity on a new commit
- [x] 4.4 Remove `gh` auth on host temporarily and verify `cage claude` still launches (graceful degradation)
