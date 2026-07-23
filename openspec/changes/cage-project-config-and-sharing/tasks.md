## 1. Spike: sbx network domain injection

- [ ] 1.1 Run `sbx run --help` (or check sbx docs) to determine if per-invocation domain allowlist flags exist
- [ ] 1.2 Document the outcome in design.md: update "Path A / Path B" with the chosen approach and any relevant sbx flag names or constraints

## 2. Implement `cage-config` helper

- [ ] 2.1 Create `cage-config` Python script: `--init <project-dir>` command creates `PROJECT/.cage/` and `PROJECT/.cage/.gitignore` if absent
- [ ] 2.2 Add `--mounts <file> [<file>...]` command: parse YAML, collect `mounts:` lists from each file (skip missing files), deduplicate, print one mount per line
- [ ] 2.3 Add `--domains <file> [<file>...]` command: parse YAML, collect `network.allowedDomains:` lists from each file (skip missing files), deduplicate, print one domain per line
- [ ] 2.4 Add `--merge-kit <kit-dir> <file> [<file>...]` command (only if spike outcome is Path B): read kit `spec.yaml`, merge extra domains into `network.allowedDomains`, write result to stdout
- [ ] 2.5 Expand `~` in all mount paths via `os.path.expanduser`; warn to stderr and skip files with invalid YAML

## 3. Update `cage` launch script

- [ ] 3.1 Call `cage-config --init "$(pwd)"` before the sbx invocation to ensure `.cage/` and `.gitignore` exist
- [ ] 3.2 Call `cage-config --mounts ~/.cage/config.yaml .cage/config.local.yaml` and incorporate the output as additional mounts in `sbx run`
- [ ] 3.3 Inject extra domains using the approach determined by the spike:
  - Path A: pass a per-invocation flag to `sbx run` for each domain
  - Path B: call `cage-config --merge-kit`, write temp kit, pass `--kit` to sbx, drop `exec` and clean up temp kit after sbx exits
- [ ] 3.4 Handle missing or empty `cage-config` output gracefully (no mounts/domains added, no error)

## 4. Update `install`

- [ ] 4.1 Add `cage-config` to the list of binaries linked onto PATH (alongside `cage` and `cage-clipd`)

## 5. Update agent-facing CLAUDE.md

- [ ] 5.1 Add a section documenting `.cage/scratch/`: purpose (persistent agent context), suggested structure (e.g. `CONTEXT.md`), and instruction to read it at session start and update it throughout
- [ ] 5.2 Add a section documenting `.cage/share/`: purpose (bidirectional file exchange with the user), when to look there (user-dropped reference files), and when to write there (outputs intended for the user)

## 6. Update README

- [ ] 6.1 Document the `.cage/` directory structure: `config.yaml`, `config.local.yaml`, `share/`, `scratch/`
- [ ] 6.2 Document the config schema with examples for mounts and network domains
- [ ] 6.3 Document the three-level merge: what lives at each level and why
- [ ] 6.4 Document `share/` as the runtime file-sharing channel (user drops files; agent writes outputs)
