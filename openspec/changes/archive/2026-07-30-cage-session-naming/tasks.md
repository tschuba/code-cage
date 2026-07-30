## 1. cage Script

- [x] 1.1 Append `-$$` to `NAME` in `cage`: change `sed 's/-$//')"` → `sed 's/-$//')-$$"`
- [x] 1.2 Delete the `sbx rm -f "$NAME" 2>/dev/null || true` line

## 2. Verification

- [x] 2.1 Start `cage` in this repo; open a second terminal in the same dir and run `cage` again — confirm first session is still alive
- [x] 2.2 Start `cage` in two dirs with the same basename (e.g. `~/a/code-cage` and `~/b/code-cage`) — confirm both run without collision
- [x] 2.3 Run `openspec apply` to merge the `cage-parallel-sessions` spec into `openspec/specs/`
