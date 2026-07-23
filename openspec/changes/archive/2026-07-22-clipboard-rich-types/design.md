## Context

`cage-clipd` is a background shell script that polls the macOS clipboard every 0.3 s and writes `~/.cage/clipboard/latest.png` when the clipboard holds an image. The cage sandbox mounts `~/.cage/clipboard/` read-only, so the agent can read the file. The WezTerm `Ctrl+Shift+V` macro hardcodes `latest.png` as the path to insert at the prompt.

The current implementation is limited to images via `pngpaste`. PDFs and files from Finder are silently ignored.

## Goals / Non-Goals

**Goals:**
- Detect PDF bytes on the pasteboard and write them as `latest.pdf`
- Detect file URLs from Finder and copy the files into `~/.cage/clipboard/`
- Apply a folder-consolidation rule: if all copied files share one parent and that parent contains exactly those files, copy into a named subdirectory and present the folder path
- Write a `.current` file on every clipboard event so the WezTerm macro is type-agnostic
- Replace `osascript md5` polling with `NSPasteboard.changeCount()`

**Non-Goals:**
- HTML clipboard selection (plain text already covers the useful part)
- RTF clipboard selection
- Deduplication or versioning of clipboard files across sessions
- Clipboard content that requires conversion (e.g. TIFF → JPEG — TIFF is converted to PNG as part of image capture, but that's handled by NSBitmapImageRep internally)

## Decisions

### Python + PyObjC instead of shell or compiled Swift

`PyObjC` ships with every macOS system Python and gives direct access to `NSPasteboard`. Shell tools (`osascript`, `pbpaste`) cannot extract PDF bytes or file URLs reliably. A compiled Swift binary would be more robust but adds a build step and a binary to the install. Python is the lazy right call here.

Alternatives: `osascript` alone (too limited), Swift CLI (more robust but unnecessary complexity for this scope).

### `NSBitmapImageRep` for image capture instead of `pngpaste`

`NSBitmapImageRep` handles every image encoding NSPasteboard can hold (PNG, TIFF, screenshots) and outputs PNG directly in Python — no subprocess, no brew dependency. `pngpaste` occasionally fails on certain TIFF encodings (some screen recordings); the AppKit path does not.

Eliminates `pngpaste` as a requirement entirely. The install script no longer needs `brew install pngpaste`.

### `.current` pointer file as the WezTerm macro target

Rather than making the WezTerm macro enumerate `latest.*` with a glob (fragile) or hardcode a new filename (same problem as before), `cage-clipd` writes a plain-text `.current` file containing the path(s) to insert — one per line. The macro becomes `cat ~/.cage/clipboard/.current`.

This decouples the macro from content type and makes multi-file paste (multiple paths, one per line) trivial.

### Folder consolidation rule for multi-file Finder paste

When multiple files are copied from Finder and they are the only contents of their parent directory, cage-clipd copies them into `~/.cage/clipboard/<foldername>/` and writes that directory path to `.current`. Otherwise files go flat into `~/.cage/clipboard/` and all paths are listed in `.current`. Files must be physically copied (not symlinked) because the container only mounts `~/.cage/clipboard/` — host paths like `~/Downloads/` don't exist inside it.

Rationale: inserting a folder path when the agent needs to reference a coherent set of documents is more natural than listing individual paths. The check is `set(os.listdir(parent)) == set(basenames)`.

### Clean clipboard dir on each new event

Old files from a previous clipboard event are removed before writing new ones. This prevents stale `latest.pdf` from being visible when the clipboard now holds an image, and avoids unbounded growth. Cleanup is scoped to `latest.*` and `.current`; the `.current` pointer is always overwritten.

## Risks / Trade-offs

- **Large file copy** — A Finder paste of large files copies them synchronously in cage-clipd. For typical document-sharing use (PDFs, office files) this is fine; a 500 MB video would block the poll loop. → Mitigation: document the expectation (documents only); add file-size guard if this becomes a problem.
- **Name collisions in flat copy** — Two files with the same basename from different directories overwrite each other. Not handled; YAGNI for this scope.
- **NSBitmapImageRep TIFF fallback** — if the pasteboard holds only a `public.png` representation (rare), the TIFF path is a no-op. The code must check both `public.tiff` and `public.png` and prefer whichever is present.

