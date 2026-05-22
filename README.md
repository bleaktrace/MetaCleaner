# MetaCleaner

A Windows tool for stripping metadata from a wide range of file types —
images, audio, video, PDFs, Word/Excel/PowerPoint (modern and legacy),
OpenDocument files, RTF, EPUB, and ZIP archives. Ships as a single
executable that runs both as a command-line tool and as a drag-and-drop
GUI. Built on top of ExifTool for breadth and format-specific Python
libraries (pikepdf, mutagen, olefile, openpyxl, python-docx, python-pptx,
Pillow, piexif) for the deeper passes ExifTool does not handle.

## Features

- 9 handler classes covering 31 file extensions.
- Atomic writes — every clean produces a temp file and only renames it
  onto the original after the handler reports success; a failed clean
  leaves the original untouched.
- Recursive cleaning of ZIP archives — a zipped `.docx` gets the full
  OOXML deep clean before being re-packed.
- Inspection mode that lists every metadata field the inspector can see,
  with a `compare(before, after)` view so you can verify what changed.
- Logger with INFO/SUCCESS/WARNING/ERROR levels, in-memory buffer,
  callback subscriptions for live UI feedback, and export to `.txt` or
  `.csv`.
- Drag-and-drop GUI on Windows via `tkinterdnd2`, with a colored log
  pane, progress bar, and a cancel button that stops between files.
- CLI with the same engine for scripted batch jobs.

## Supported file types

| Category      | Extensions |
|---------------|-----------|
| Legacy Office | `.doc`, `.xls`, `.ppt` |
| OOXML         | `.docx`, `.xlsx`, `.pptx` |
| OpenDocument  | `.odt`, `.ods`, `.odp` |
| PDF           | `.pdf` |
| Images        | `.jpg`, `.jpeg`, `.png`, `.tiff`, `.tif`, `.heic`, `.gif`, `.bmp`, `.webp`, `.raw`, `.cr2`, `.nef` |
| Audio         | `.mp3`, `.wav`, `.flac` |
| Video         | `.mp4`, `.mov`, `.avi` |
| Archives      | `.zip` (with recursive in-archive cleaning) |
| Other         | `.rtf`, `.epub` |

## Installation

### End users — pre-built executable

1. Download `metacleaner-gui.exe` (for double-click drag-and-drop use) or
   `metacleaner.exe` (for command-line use) from the `dist/` folder of a
   release build.
2. Place it anywhere on your system. No installer, no registry changes.
3. Antivirus heuristics sometimes flag PyInstaller bundles. If your AV
   blocks the file, add an exclusion or re-run a build from source.

## CLI usage

```
metacleaner <path>                      Clean a file or directory
metacleaner <path> --inspect            Show metadata, do not modify
metacleaner <path> --recursive          Recurse into subdirectories (default)
metacleaner <path> --no-recursive       Process only the top level
metacleaner <path> --output <dir>       Write cleaned files under <dir>
metacleaner <path> --log run.csv        Export the run log as CSV
metacleaner <path> --log run.txt        Export the run log as plain text
metacleaner <path> --no-zip-recurse     For .zip files, clean only archive
                                        metadata — do not touch the contents
metacleaner --list-formats              Print supported extensions
metacleaner --version                   Print program version
```

Examples:

```cmd
:: Inspect a single file
metacleaner photos\IMG_0001.jpg --inspect

:: Clean one file in place
metacleaner photos\IMG_0001.jpg

:: Clean a folder into a sibling output folder, preserving structure
metacleaner photos\ --output photos_clean

:: Clean an archive but leave the contents alone
metacleaner backup.zip --no-zip-recurse

:: Write a CSV log alongside the run
metacleaner reports\ --log run.csv
```

Per-file output is one line:

```
[OK]   path/to/file.docx — 12 fields removed
[WARN] path/to/file.mp3 — 5 fields removed, 1 warning: ...
[SKIP] path/to/file.zzz — unsupported extension
[FAIL] path/to/file.bad — ...error...
```

Exit codes: `0` on success (warnings and skips do not count as failure),
`1` if any file failed outright, `2` on argument errors.

## GUI usage

```
+--------------------------------------------------+
|               drop zone                          |
|   "Drag files or folders here ..."               |
+--------------------------------------------------+
|  Add Files...  Add Folder...  Clear List         |
+--------------------------------------------------+
|   queued items (Path | Type | Status)            |
+--------------------------------------------------+
|   [x] Recursive   [x] Clean inside ZIP           |
|   ( ) Overwrite   ( ) Save to: [path] [Browse]   |
+--------------------------------------------------+
|   Inspect Selected   Clean All   Stop            |
+--------------------------------------------------+
|   colored log Text widget (scrollable)           |
+--------------------------------------------------+
|   [progress bar]   Ready                Export...|
+--------------------------------------------------+
```

Workflow:

1. Drag files or folders into the drop zone, or use **Add Files...** /
   **Add Folder...**.
2. Tick **Recursive** if you want subdirectories descended.
3. Choose **Overwrite original** or **Save to...** an output folder.
4. Click **Inspect Selected** to preview what metadata will be removed,
   or **Clean All** to run the queue.
5. Click **Stop** to break out of the loop between files.
6. Use **Export Log...** to save the colored log to `.txt` or `.csv`.

## What gets removed

| Format              | Removed |
|---------------------|---------|
| OOXML (.docx/.xlsx/.pptx) | Every `dc:*` and `cp:*` field in core.xml (creator, title, subject, keywords, lastModifiedBy, created, modified, revision); leak fields in app.xml (Company, Manager, HyperlinkBase, Template, TotalTime); Application and AppVersion normalised to neutral values; docProps/custom.xml dropped wholesale; docProps/thumbnail.* dropped; .docx comment/people parts dropped with matching [Content_Types] and .rels pruning; `w:author`/`w:date`/`w:rsid*` attributes stripped throughout document.xml/headers/footers/styles; `<w:rsids>` block removed from settings.xml; xlsx comment authors blanked; xlsx persons displayName blanked; pptx commentAuthor name/initials blanked; pptx notesSlides author/date attrs stripped |
| OpenDocument (.odt/.ods/.odp) | Every child of `office:meta` except `meta:document-statistic` (word counts kept); `Thumbnails/` dropped; settings.xml `config-item` entries with names containing `author`/`user`/`lastsave`/`savedby` removed |
| Legacy Office (.doc/.xls/.ppt) | `\005SummaryInformation` and `\005DocumentSummaryInformation` PropertySet streams cleared by ExifTool; olefile-based residual check logs WARNING for anything left |
| PDF (.pdf) | `/Info` dictionary cleared; XMP `/Metadata` stream removed; `/Outlines` dropped; `/Names/JavaScript` actions dropped; ExifTool secondary pass runs as belt-and-braces |
| Images | All EXIF (camera make/model/software/artist/copyright), all GPS, all IPTC, all XMP, JFIF artifacts, PNG tEXt/zTXt/iTXt chunks; ICC_Profile and colour-space tags deliberately preserved so the image still renders correctly |
| Audio (.mp3) | Every ID3v1 and ID3v2 tag (Title, Artist, Album, Genre, Year, Comments, ...) via mutagen — ExifTool currently has no MP3 writer |
| Audio/Video (.wav/.flac/.mp4/.mov/.avi) | All metadata atoms/chunks/vorbis comments via ExifTool |
| RTF (.rtf) | `{\info ...}`, `{\*\userprops ...}`, `{\*\generator ...}`, `{\*\password ...}`, `{\*\passwordhash ...}`, `{\*\protend ...}`, `{\*\protstart ...}` brace groups; `\revauth`, `\revdttm`, `\edmins`, `\revtim`, `\creatim`, `\printim` control words |
| EPUB (.epub) | Every `<dc:*>` element's text cleared; `<dc:identifier>` replaced with a fresh UUID; calibre and `dcterms:*` `<meta>` entries dropped; `META-INF/calibre_bookmarks.txt` dropped |
| ZIP (.zip) | Archive comment cleared; per-entry comments cleared; per-entry timestamps normalised to `(1980,1,1,0,0,0)`; `create_system` normalised to MS-DOS; `external_attr` reduced to the DOS archive/directory bit; per-entry extras filtered to keep only ZIP64 (`0x0001`) and UTF-8 path (`0x7075`); contained files passed through the matching handler recursively |

## Known limitations

- **Hidden xlsx sheets and hidden pptx slides are preserved** — removing
  them would discard user content. We log a WARNING listing each one so
  you can decide whether to delete them by hand.
- **VBA macros are preserved** — same reasoning. Embedded macro projects
  can carry authoring metadata that is not feasible to scrub without
  rewriting the macro binary, and we log a WARNING for every `*/vbaProject.bin`
  encountered.
- **Legacy `.doc`/`.xls`/`.ppt`** rely on ExifTool's OLE writer.
  Uncommon properties can survive — the handler runs olefile's
  `get_metadata()` after the ExifTool pass and emits a WARNING for any
  residual field. For sensitive material, re-run `--inspect` afterward
  to confirm.
- **MP3 files** are cleaned by mutagen because ExifTool 13.58 explicitly
  rejects writing to MP3 (`Writing of MP3 files is not yet supported`).
  The handler logs that as a WARNING and continues with mutagen's
  ID3v1/v2 strip.
- **Filename**, **file timestamps**, and **filesystem attributes** are not
  touched (they are not "inside" the file). Rename and `touch` manually
  if you need those clean too.


## Troubleshooting

- **"Windows protected your PC" / SmartScreen warning** — PyInstaller
  binaries are unsigned; SmartScreen flags them until enough users have
  vouched for the hash. Click "More info" → "Run anyway", or sign the
  binary with your own code-signing certificate.
- **Antivirus quarantines the exe** — false positive triggered by the
  PyInstaller bootloader. Add an exclusion for `metacleaner.exe` and
  `metacleaner-gui.exe`, or rebuild from source on the affected machine.
- **First run is slow** — the one-file bundle extracts itself to a temp
  directory. Subsequent launches reuse that extraction and are fast.
- **`No handler for extension '.xyz'`** — `metacleaner.exe --list-formats`
  prints exactly what is supported. Anything outside that list is
  passed through untouched.
