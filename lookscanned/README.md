# LookScanned portable CLI

Turn PDFs into scanned-looking image PDFs, entirely offline. The Windows x64
release includes its own runtime: no Python, Node, Electron, Git, or installer is
needed on the receiving PC. Unzip the release, keep its folder together, and run:

```powershell
.\lookscanned\bin\lookscanned.exe "input.pdf" -o "output.pdf" --preset subtle --seed 42
```

The source version is `python cli/lookscanned.py ...`. Output defaults to
`INPUT-scan.pdf`. Existing output requires `--overwrite`; the input itself can
never be overwritten. Quote paths containing spaces. Output folders must exist.
PDF text, links, forms, and bookmarks become page images; this tool does not OCR.

## GUI settings

Defaults match the GUI's Canvas scanner. Every slider range is supported,
including intermediate values; CLI values need not be snapped to slider steps.

| GUI control | CLI option | Range / choices | Default | GUI step |
|---|---|---|---|---|
| Colorspace | `--colorspace` | `gray`, `sRGB` | `gray` | switch |
| Border | `--border`, `--no-border` | on / off | off | switch |
| Rotate | `--rotate` | -10 to 10 degrees, clockwise positive | 1 | 0.1 |
| Rotate variance | `--rotate-variance` (alias `--rotate-var`) | 0 to 10 degrees, symmetric +/- | 0.5 | 0.1 |
| Brightness | `--brightness` | 0 to 2; 1 neutral | 1 | 0.01 |
| Yellowish | `--yellowish` | 0 to 2; 0 neutral | 0 | 0.01 |
| Contrast | `--contrast` | 0 to 2; 1 neutral | 1 | 0.01 |
| Blur | `--blur` | 0 to 1 raster pixels | 0.3 | 0.01 |
| Noise | `--noise` | 0 to 1; 0 disables grain | 0.1 | 0.01 |
| Resolution | `--scale` | 1 to 3 | 2 | 0.5 |

`--dpi` is an alternative to `--scale`: DPI = scale * 72. GUI resolution choices
are 72, 108, 144, 180, and 216 DPI. Physical page dimensions stay unchanged.
Use `--output-format png` for lossless page images or `jpeg` (default) for smaller
PDFs. `--jpeg-quality 1..100` defaults to 92 and has no effect on PNG output.
The GUI has an internal output-format setting, but no visible format control.

The CLI reproduces the controls and purpose, not exact browser pixels:

- PDFium renders input pages instead of PDF.js.
- Variance follows the GUI's +/- label. The existing Canvas renderer actually
  varies only in the positive direction; the Magica renderer is symmetric.
- Noise is seeded monochrome Gaussian grain, not the Canvas SVG texture. Zero
  really disables it (the Magica implementation has a zero-noise discrepancy).
- Yellowish blends toward cream RGB(252,242,199), 20% per unit (40% at 2), like
  the Magica control. Canvas uses sepia and saturates its effect at 1.
- Effects run as blur, colorspace, brightness, tint, contrast, rotation, grain,
  then an optional one-pixel black frame. The frame stays aligned with the page.
- Rotation keeps the page rectangle; content near edges can be clipped.

## Presets and automation

| Preset | Purpose | Values different from GUI defaults |
|---|---|---|
| `gui` | GUI defaults | none |
| `clean` | Rasterize with no scan effects | sRGB, rotation/variance/blur/noise 0 |
| `subtle` | Light scan texture | rotation 0, variance 0.3, blur 0.2, noise 0.08 |
| `aged` | Visible wear and paper tint | rotation 0, variance 1.2, blur 0.5, noise 0.25, yellowish 0.65, brightness 0.97, contrast 0.95 |

Explicit flags always override the preset, regardless of argument order.

```powershell
# Tune a few pages before processing a long document.
.\lookscanned\bin\lookscanned.exe "input.pdf" -o "preview.pdf" --pages 2-4 --preset aged --noise 0.15 --seed 42 --json

# Select color output and a specific resolution.
.\lookscanned\bin\lookscanned.exe --input "input.pdf" -o "color.pdf" --colorspace sRGB --dpi 180 --rotate 0 --rotate-variance 0 --seed 42
```

`--pages` accepts 1-based comma-separated pages/ranges, removes duplicates, and
keeps document order. Without it, all pages are processed. `--password` opens an
encrypted input; the output is not encrypted. `--seed` accepts 0 through 2^64-1.
An omitted seed is generated and reported. With the same input, settings, seed,
and tool version, results are reproducible; selecting a page does not change its
random effects relative to a full-document run.

`--json` prints one UTF-8 JSON object on stdout. Success includes `ok`, `output`,
`pages`, `source_pages`, `settings`, `seed`, `page_angles`, `bytes`, and timing.
Failures contain `ok: false`, `error`, and `exit_code`. Progress goes to stderr;
`--quiet` suppresses it. `--help` and `--version` remain human-readable.
Exit codes: 0 success, 2 invalid arguments/paths, 1 PDF or I/O error, 130 cancelled.

Pages are rendered sequentially; compressed page data still accumulates while
the output PDF is assembled. A 20-million-pixel per-page limit prevents extreme
allocations. Reduce resolution for unusually large pages. Output is published
only after successful completion; partial temporary files are removed on handled
errors or Ctrl+C. A forced process kill or machine shutdown can leave a temporary
file named `.lookscanned-*.tmp` in the output directory.

## Use as a Codex skill

The ZIP contains a complete `lookscanned` skill folder. Copy that whole folder
into your Codex skills directory (normally `%USERPROFILE%\.codex\skills`). Keep
`SKILL.md`, `bin`, and the bundled files together. The skill runs its own executable
by absolute path; it does not depend on a repository checkout or PATH entry.
Restart/reload skill discovery if the current chat does not yet list the skill.

## Development and build

Use a project environment, not base Python. On this machine use `codex_env`.
Python 3.12 x64 is the tested build runtime.

```powershell
conda run -n codex_env python -m pip install -r cli/requirements.txt -r cli/requirements-build.txt
conda run -n codex_env python -B -m unittest discover -s cli/tests -v
conda run -n codex_env python -B cli/build_windows.py
```

The build produces `release/lookscanned-windows-x64.zip`. It cleans its temporary
PyInstaller work directories. Use the build script's `--overwrite` option when
intentionally replacing an existing release. Runtime dependencies and bundled
third-party license texts are included. This is a Windows x64 build, not a single
binary for every operating system or CPU architecture.

To run the same behavior tests against an extracted executable:

```powershell
$env:LOOKSCANNED_TEST_EXE = 'C:\path\lookscanned\bin\lookscanned.exe'
conda run -n codex_env python -B -m unittest discover -s cli/tests -v
Remove-Item Env:LOOKSCANNED_TEST_EXE
```

Tests generate their own PDFs in temporary directories and remove them on exit.
The build and tests do not change the original Vue/Electron application.
