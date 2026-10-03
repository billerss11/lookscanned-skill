# LookScanned CLI options

These values describe the bundled **native CLI 1.0.0**, checked against its
parser, preset definitions, and image-processing code. Browser appearance differs.

## Defaults and presets

The table below uses **CLI `gui` preset defaults**, also used when `--preset` is omitted.
The ten GUI controls use Canvas defaults; DPI and JPEG quality are CLI additions.
The skill normally selects **`subtle`**. A preset supplies starting values;
explicit flags always override it, regardless of argument order.

| Preset | Purpose | Changes from `gui`; all other values inherited |
|---|---|---|
| `gui` | GUI's Canvas defaults | None |
| `clean` | Color rasterization, no scan effects | colorspace `sRGB`; rotate, variance, blur, noise `0` |
| `subtle` | Light scanner effect | rotate `0`; variance `0.3`; blur `0.2`; noise `0.08` |
| `aged` | Visible wear and paper tint | rotate `0`; variance `1.2`; blur `0.5`; noise `0.25`; yellowish `0.65`; brightness `0.97`; contrast `0.95` |

## Image controls

Ranges are inclusive. Fractions are allowed except for JPEG quality. CLI values
need not follow the GUI slider steps. Directions below hold other controls fixed;
clipping and later effects can alter the final appearance.

| Option | CLI `gui` default | Minimum .. maximum / choices | What increasing or decreasing does |
|---|---|---|---|
| `--colorspace` | `gray` | `gray`, `sRGB` | Categorical: `gray` converts to grayscale; `sRGB` retains input color. Gray is not two-tone black/white. Later tint can add color. |
| `--border` / `--no-border` | off | off, on | Adds/removes a one-raster-pixel black frame, drawn last. More resolution makes this fixed-pixel frame physically thinner. |
| `--rotate` | `1` | `-10 .. 10` degrees | Signed base angle: positive clockwise, negative counterclockwise. Moving toward zero reduces base tilt; increasing a negative value does not mean more tilt. Zero still permits variance. |
| `--rotate-variance` / `--rotate-var` | `0.5` | `0 .. 10` degrees | Larger widens random deviation on both sides of the base angle; smaller narrows it; zero uses the base exactly. A particular page can move closer to level even with larger variance. |
| `--brightness` | `1` | `0 .. 2` | Multiplies channels: lower darkens, higher brightens and may clip highlights. `1` unchanged; `0` black at this stage. Later tint/contrast/noise can lift black. |
| `--yellowish` | `0` | `0 .. 2` | Higher blends more toward cream RGB(252,242,199), 20% per unit, up to 40%; lower reduces that blend. It lightens dark pixels and darkens white, not simply darkening all pixels. Zero disables tint. |
| `--contrast` | `1` | `0 .. 2` | Higher pushes values away from middle gray: dark darker, light lighter, with possible clipping. Lower pulls both toward gray. `1` unchanged; `0` middle gray before subsequent effects. |
| `--blur` | `0.3` | `0 .. 1` raster pixels | Larger Gaussian radius softens edges more; smaller softens less; zero disables added blur. Lowering it cannot restore detail absent from the input. |
| `--noise` | `0.1` | `0 .. 1` | Higher increases monochrome Gaussian grain amplitude; lower decreases it; zero disables grain. This controls variation, not a uniform brightness shift. Pixel clipping can bias very light/dark areas. |
| `--scale` | `2` | `1 .. 3` | Higher renders more pixels; lower fewer. Twice the scale gives about four times the raster pixels and more processing/memory. It may retain more fine detail but cannot recover missing input detail. Paper size stays fixed. |
| `--dpi` | `144` effective | `72 .. 216` | Alternative to scale: `DPI = scale * 72`. Same increase/decrease effects. **Do not combine with `--scale`.** |
| `--output-format` | `jpeg` | `jpeg`, `png` | JPEG adds lossy compression; PNG embeds lossless raster images, usually larger. PNG does not preserve vector text or undo earlier effects. |
| `--jpeg-quality` | `92` | integer `1 .. 100` | Higher requests less aggressive JPEG compression, usually fewer artifacts/larger files; lower the reverse. File size/error are not guaranteed strictly monotonic for every image. `100` is not lossless. Ignored for PNG. |

Actual angle is `rotate + uniform(-variance, +variance)`. Thus the combined
angle is bounded by -20 and +20 degrees. Rotation keeps the page rectangle,
fills exposed areas white, and can clip edge content.

Effects run **blur -> grayscale (if selected) -> brightness -> cream tint ->
contrast -> rotation -> grain -> border**. For example, `gray` plus nonzero tint
produces colored paper; `contrast 0` can still show later grain/frame. `clean`
is a useful baseline for isolating one control. Blur is measured in raster pixels,
so its physical softness also depends on resolution.

## Files, automation, and other options

| Argument / option | Default | Accepted values and behavior |
|---|---|---|
| `INPUT` or `--input` | required | Existing PDF path; supply exactly one form, not both. Quote paths containing spaces. |
| `-o` / `--output` | sibling `<input-stem>-scan.pdf` | Drops the input's final extension, e.g. `report.pdf` -> `report-scan.pdf`. Requires a distinct `.pdf` path in an existing directory. Never replaces the input, even with overwrite. |
| `--preset` | `gui` | `gui`, `clean`, `subtle`, `aged`; see preset table. |
| `--seed` | generated and reported | Integer `0 .. 18446744073709551615`. Changes the random pattern, **not strength**; larger is not stronger. Same input, selected pages, settings, seed, and runtime environment reproduce output. |
| `--pages` | all pages | 1-based pages/ranges, e.g. `1-3,5`, bounded by the input's page count. Duplicates removed; output stays in document order. Selecting more pages adds work, not quality. A selected page gets the same effects as in a full run with that seed. |
| `--password` | none | Unlocks an encrypted input. Output is not encrypted. |
| `--overwrite` | off | Allows replacement of an existing output. Without it, existing outputs produce an error. |
| `--json` | off | One UTF-8 result on stdout, including errors. Success includes `ok`, `output`, `pages`, `settings`, `seed`, `bytes`, timing and per-page arrays. Keep only useful fields for routine agent output. |
| `--quiet` | off | Suppresses per-page stderr progress. Does not suppress the final result or change PDF quality. Use with `--json` for normal agent calls. |
| `-h` / `--help` | off | Prints help and exits; no input required. |
| `--version` | off | Prints version and exits; no input required. |

`--help`/`--version` stay human-readable even with `--json`. Exit codes: `0`
success; `2` argument/explicit path validation or pixel-limit error; `1` PDF,
OS/I/O (including permissions or temporary-file failures), or memory error; `130`
cancelled. JSON errors contain `ok: false`, `error`, and `exit_code`.

## Long documents and differences from the GUI

See [measured tests](validation.md) for the successful 314-page run and limits.
There is no fixed page-count cap. Pages render sequentially, but the PDF writer
retains compressed page data until saving: memory still grows with the document.
Each page is limited to 20 million raster pixels. Higher resolution and noisy or
image-heavy pages can increase memory/output size substantially; hundreds of
pages are not a guarantee for every PDF or machine. On failure, report it and
offer lower resolution or a requested subset without silently changing the job.
Handled errors/Ctrl+C remove partial output; forced termination can leave a
`.lookscanned-*.tmp` file in the output directory.

The original Canvas GUI uses PDF.js, SVG texture, CSS sepia, and one-sided
rotation variance. Its Magica fallback uses different filters and symmetric
variance. This CLI uses PDFium, Gaussian grain, and cream blending. The exposed
controls/ranges are comparable; exact pixels and parameter strengths are not
interchangeable across those engines.
