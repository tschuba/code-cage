## Requirements

### Requirement: Clipboard daemon syncs PDF bytes to sandbox
When the macOS clipboard holds a PDF (UTI `com.adobe.pdf`), `cage-clipd` SHALL write the raw PDF bytes to `$TMPDIR/cage-clipboard-<YYYYMMDD-HHMMSS>.pdf` and update `$TMPDIR/cage-clipboard.current` to that path.

#### Scenario: PDF copied from Preview
- **WHEN** the user copies a page or selection in Preview (clipboard gains `com.adobe.pdf` type)
- **THEN** a file matching `$TMPDIR/cage-clipboard-<timestamp>.pdf` is written with the PDF bytes
- **THEN** `$TMPDIR/cage-clipboard.current` contains the single line pointing to that file

#### Scenario: PDF from browser
- **WHEN** the user copies PDF content from a browser (clipboard gains `com.adobe.pdf` type)
- **THEN** a timestamped PDF file is written to `$TMPDIR`
- **THEN** `$TMPDIR/cage-clipboard.current` is updated to point to it

#### Scenario: Non-PDF clipboard event does not remove prior PDF
- **WHEN** the clipboard changes to a non-PDF, non-image type after a PDF was synced
- **THEN** the prior PDF file in `$TMPDIR` is left in place (OS handles cleanup); only `.current` is updated or cleared

### Requirement: Clipboard daemon copies Finder files to sandbox
When the macOS clipboard holds file URLs (UTI `NSFilenamesPboardType`), `cage-clipd` SHALL copy each file to `$TMPDIR` with a `cage-clipboard-<timestamp>-<originalname>` prefix and write the resulting path(s) to `$TMPDIR/cage-clipboard.current`, one path per line.

#### Scenario: Single file copied from Finder
- **WHEN** the user copies one file in Finder (e.g. `~/Downloads/report.pdf`)
- **THEN** the file is copied to `$TMPDIR/cage-clipboard-<timestamp>-report.pdf`
- **THEN** `$TMPDIR/cage-clipboard.current` contains that path

#### Scenario: Multiple files from different locations
- **WHEN** the user copies multiple files from different directories
- **THEN** each file is copied to `$TMPDIR/cage-clipboard-<timestamp>-<basename>`
- **THEN** `$TMPDIR/cage-clipboard.current` contains one path per line for each copied file

#### Scenario: Multiple files constituting their entire parent folder
- **WHEN** the user copies N files from a directory and those N files are the only contents of that directory
- **THEN** the files are copied into `$TMPDIR/cage-clipboard-<timestamp>-<foldername>/`
- **THEN** `$TMPDIR/cage-clipboard.current` contains the single line pointing to that directory

### Requirement: Current pointer always reflects latest clipboard event
`cage-clipd` SHALL overwrite `$TMPDIR/cage-clipboard.current` on every clipboard event that produces output. Prior output files are NOT removed — the OS handles `$TMPDIR` cleanup automatically.

#### Scenario: Image after PDF
- **WHEN** the clipboard changes from a PDF to an image
- **THEN** a new `cage-clipboard-<timestamp>.png` is written to `$TMPDIR`
- **THEN** `.current` is updated to point to the new image
- **THEN** the prior PDF file remains in `$TMPDIR` (not deleted)

### Requirement: WezTerm paste macro inserts current clipboard path(s)
The `Ctrl+Shift+V` WezTerm keybinding SHALL insert the contents of `$TMPDIR/cage-clipboard.current` at the cursor, which may be one or more paths separated by newlines.

#### Scenario: Single file path inserted
- **WHEN** `.current` contains one path and the user presses `Ctrl+Shift+V`
- **THEN** that path is inserted at the cursor position in the terminal prompt

#### Scenario: Multiple paths inserted for multi-file paste
- **WHEN** `.current` contains multiple paths and the user presses `Ctrl+Shift+V`
- **THEN** all paths are inserted, one per line, at the cursor position
