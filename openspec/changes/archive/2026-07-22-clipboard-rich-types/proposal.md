## Why

`cage-clipd` currently bridges only PNG images from the host clipboard into the cage sandbox. Users frequently need to share PDFs (from Preview or browser) and arbitrary files (from Finder) with the cage agent as context — there is no supported path for this today.

## What Changes

- `cage-clipd` is rewritten in Python (using PyObjC / AppKit) to detect clipboard type and handle multiple content types
- PDF bytes on the clipboard are written to `~/.cage/clipboard/latest.pdf`
- Files copied from Finder are physically copied into `~/.cage/clipboard/`; if all files share a parent directory and that directory contains exactly the copied files, they are placed in a named subdirectory matching the parent folder name
- A `~/.cage/clipboard/.current` file is written on every clipboard change, containing the path(s) the agent should use (one per line)
- The WezTerm `Ctrl+Shift+V` macro is updated to read and insert `.current` instead of hardcoding `latest.png`
- Image capture moves to `NSBitmapImageRep`; `pngpaste` dependency is dropped entirely
- `NSPasteboard.changeCount()` replaces the current `osascript md5` polling

## Capabilities

### New Capabilities

- `clipboard-bridge`: Syncing non-text host clipboard content (PDF bytes, Finder files) into the cage sandbox at `~/.cage/clipboard/`, with a `.current` pointer that the WezTerm macro reads to insert the right path(s) at the prompt

### Modified Capabilities

*(none)*

## Impact

- `cage-clipd`: full rewrite (shell → Python)
- `install`: update install steps and dependency notes
- `README.md`: update "Pasting screenshots" section to cover PDFs and files
- WezTerm config snippet: update `Ctrl+Shift+V` macro to read `.current`
- No changes to `cage`, `sbx` invocation, or any agent kit
