## ADDED Requirements

### Requirement: Clipboard daemon syncs PDF bytes to sandbox
When the macOS clipboard holds a PDF (UTI `com.adobe.pdf`), `cage-clipd` SHALL write the raw PDF bytes to `~/.cage/clipboard/latest.pdf` and set `.current` to that path.

#### Scenario: PDF copied from Preview
- **WHEN** the user copies a page or selection in Preview (clipboard gains `com.adobe.pdf` type)
- **THEN** `~/.cage/clipboard/latest.pdf` is written with the PDF bytes
- **THEN** `~/.cage/clipboard/.current` contains the single line `~/.cage/clipboard/latest.pdf`

#### Scenario: PDF from browser
- **WHEN** the user copies PDF content from a browser (clipboard gains `com.adobe.pdf` type)
- **THEN** `~/.cage/clipboard/latest.pdf` is written with the PDF bytes
- **THEN** `~/.cage/clipboard/.current` contains the single line `~/.cage/clipboard/latest.pdf`

#### Scenario: Non-PDF clipboard event clears stale PDF
- **WHEN** the clipboard changes to a non-PDF, non-image type after a PDF was synced
- **THEN** `~/.cage/clipboard/latest.pdf` is removed

### Requirement: Clipboard daemon copies Finder files to sandbox
When the macOS clipboard holds file URLs (UTI `NSFilenamesPboardType`), `cage-clipd` SHALL copy each file to `~/.cage/clipboard/` and write the resulting path(s) to `.current`, one path per line.

#### Scenario: Single file copied from Finder
- **WHEN** the user copies one file in Finder (e.g. `~/Downloads/report.pdf`)
- **THEN** the file is copied to `~/.cage/clipboard/report.pdf`
- **THEN** `~/.cage/clipboard/.current` contains `~/.cage/clipboard/report.pdf`

#### Scenario: Multiple files from different locations
- **WHEN** the user copies multiple files from different directories
- **THEN** each file is copied flat into `~/.cage/clipboard/<basename>`
- **THEN** `~/.cage/clipboard/.current` contains one path per line for each copied file

#### Scenario: Multiple files constituting their entire parent folder
- **WHEN** the user copies N files from a directory and those N files are the only contents of that directory
- **THEN** the files are copied into `~/.cage/clipboard/<foldername>/`
- **THEN** `~/.cage/clipboard/.current` contains the single line `~/.cage/clipboard/<foldername>/`

### Requirement: Current pointer always reflects latest clipboard event
`cage-clipd` SHALL write `~/.cage/clipboard/.current` on every clipboard event that produces output, and remove stale files from prior events before writing new ones.

#### Scenario: Image after PDF
- **WHEN** the clipboard changes from a PDF to an image
- **THEN** `latest.pdf` is removed
- **THEN** `latest.png` is written
- **THEN** `.current` is updated to `~/.cage/clipboard/latest.png`

#### Scenario: New event clears old Finder files
- **WHEN** a new clipboard event occurs after Finder files were synced
- **THEN** previously copied files in `~/.cage/clipboard/` are removed before the new content is written

### Requirement: WezTerm paste macro inserts current clipboard path(s)
The `Ctrl+Shift+V` WezTerm keybinding SHALL insert the contents of `~/.cage/clipboard/.current` at the cursor, which may be one or more paths separated by newlines.

#### Scenario: Single file path inserted
- **WHEN** `.current` contains one path and the user presses `Ctrl+Shift+V`
- **THEN** that path is inserted at the cursor position in the terminal prompt

#### Scenario: Multiple paths inserted for multi-file paste
- **WHEN** `.current` contains multiple paths and the user presses `Ctrl+Shift+V`
- **THEN** all paths are inserted, one per line, at the cursor position
