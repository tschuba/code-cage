## 1. Rewrite cage-clipd

- [x] 1.1 Replace shell script with Python script using PyObjC: poll via `NSPasteboard.changeCount()`, detect `com.adobe.pdf` type, write `latest.pdf`
- [x] 1.2 Handle `NSFilenamesPboardType`: copy file(s) to `~/.cage/clipboard/`, apply folder-consolidation rule (single subdir if files are the sole contents of their parent), write all paths to `.current`
- [x] 1.3 Handle image types using `NSBitmapImageRep`: read `public.tiff` (or `public.png`) from pasteboard, render to PNG, write `latest.png` — no pngpaste subprocess
- [x] 1.4 Write `.current` on every successful clipboard event; clean stale `latest.*` files and prior Finder copies before each new write

## 2. Update WezTerm macro

- [x] 2.1 Change the `Ctrl+Shift+V` keybinding action from inserting a hardcoded string to reading and inserting the contents of `~/.cage/clipboard/.current`

## 3. Update install and documentation

- [x] 3.1 Update `install` script: remove `pngpaste` brew install step, verify Python 3 is available (macOS 12+ ships it), update printed WezTerm snippet to reflect new macro
- [x] 3.2 Update `README.md`: remove `pngpaste` from Requirements, expand "Pasting screenshots" section to document PDF and Finder file paste, update WezTerm config snippet
