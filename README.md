# MetaCleaner

MetaCleaner — a Windows tool that removes metadata from documents, images,
audio, video, and archives. Runs fully offline; no installation required.

## Features

- Deep metadata removal across 69 file types
- Both a drag-and-drop GUI and a command-line interface
- Inspect files before/after cleaning (see exactly what metadata exists)
- Batch processing of files and folders, preserving folder structure
- Original files never corrupted (atomic write with validation)
- Runs entirely offline — nothing is uploaded anywhere

## Supported formats

`FULL` = all standard metadata removed | `PARTIAL` = some may remain |
`INSPECT-ONLY` = viewable but not writable with bundled tools.

| Category | Depth | Extensions |
|----------|-------|-----------|
| Legacy Office | FULL | `.doc` `.xls` `.ppt` |
| OOXML | FULL | `.docx` `.xlsx` `.pptx` |
| OOXML macro-enabled | FULL | `.docm` `.xlsm` `.pptm` (macros kept) |
| OOXML templates | FULL | `.dotx` `.dotm` `.xltx` `.xltm` `.potx` `.potm` |
| ODF | FULL | `.odt` `.ods` `.odp` `.odg` |
| iWork | PARTIAL | `.pages` `.numbers` `.key` |
| PDF | FULL | `.pdf` |
| Images | FULL | `.jpg` `.jpeg` `.png` `.tiff` `.tif` `.gif` `.bmp` `.webp` `.heic` `.heif` `.avif` `.jxl` |
| RAW images | FULL | `.raw` `.cr2` `.nef` `.arw` `.dng` `.orf` `.rw2` `.raf` `.srw` `.pef` `.rwl` |
| Vector | FULL | `.svg` `.svgz` |
| Audio | FULL | `.mp3` `.flac` `.m4a` `.ogg` `.opus` `.wma` `.wav` `.aiff` `.aif` `.aifc` |
| Video | PARTIAL | `.mp4` `.mov` `.m4v` `.wmv` `.avi` |
| Video | INSPECT-ONLY | `.mkv` `.webm` `.flv` |
| Archives | FULL | `.zip` |
| Other | FULL | `.rtf` `.epub` |

The exact list is always available via `metacleaner.exe --list-formats`.

## Installation & running

1. Download the release ZIP and extract it.
2. **Keep all files together** in the extracted `MetaCleaner` folder. Do NOT
   move `metacleaner-gui.exe` out on its own — it needs the `_internal` and
   `tools` folders next to it.
3. Run **`metacleaner-gui.exe`** for the graphical interface (double-click).
4. Run **`metacleaner.exe`** from a command prompt for CLI use.

No Python installation required.

## CLI usage

```
metacleaner.exe <path>                Clean a file or folder
metacleaner.exe <path> --inspect      Show metadata without changing the file
metacleaner.exe <path> --recursive    Recurse into subfolders
metacleaner.exe --list-formats        List supported formats
metacleaner.exe --help                Full options
```

## Notes / Troubleshooting

- **Antivirus false positives:** tools packaged this way (PyInstaller) are
  sometimes flagged as a false positive. If needed, add an exclusion for the
  MetaCleaner folder.
- **Corporate / managed computers:** if the app is blocked by an
  administrator policy (AppLocker / Group Policy), ask your IT administrator
  to allow the MetaCleaner folder, or specifically `metacleaner-gui.exe`,
  `metacleaner.exe`, and `tools\exiftool\exiftool.exe`.
- **Partial / inspect-only formats:** some video formats can only be
  partially cleaned or inspected (see the format list). Verify with
  `--inspect` after cleaning when working with sensitive files.

## License

MIT — see [LICENSE](LICENSE).
