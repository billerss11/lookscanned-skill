# Parameter and long-document validation

Verified on 2026-10-03. This documentation update does not change the scanner
algorithm or executable. The bundled CLI remains version 1.0.0.

## Code cross-check and tests

The option reference was checked against `cli/lookscanned.py` (`Settings`,
`PRESETS`, `build_parser`, `apply_effects`, `scan_pdf`) and the original project's
Canvas/Magica scanners, GUI slider limits, and PDF export loop.

**23 source tests passed**, including eight focused parameter-semantics tests:

| Parameter | Controlled evidence |
|---|---|
| Brightness | 0 produces black at that stage; 0.5 halves channel values; 2 doubles them with clipping at 255. |
| Contrast | Values 64/128/192 become 95/127/159 at 0.5, all 127 at 0, and 0/128/255 at 2. |
| Yellowish | At 1, black becomes RGB(50,48,39), proving dark areas can lighten. Increasing 1 to 2 moves samples closer to cream; grayscale plus tint becomes colored. |
| Blur | Checkerboard edge detail decreases from radius 0 to 0.5 to 1. |
| Noise and seed | Zero adds no grain; larger noise increases measured dispersion. Same seed repeats the pattern; a much larger seed changes it without systematically increasing strength. |
| Rotation | A marker above center moves right for +10 degrees and left for -10, confirming clockwise-positive direction. |
| Variance | Zero is fixed at the base angle. Same seeded draws at 2 and 10 span both signs and have wider dispersion at 10. |
| JPEG quality | On a fixed CLI fixture, quality 95 has lower pixel error versus a PNG reference and larger output than quality 5. This is not a universal size/error monotonicity guarantee. |

The existing behavior tests also verify grayscale/color, border, min/max accepted
values, actual embedded image dimensions at different scale/DPI values, unchanged
physical page sizes, PNG/JPEG encoding, selection order, seeded repetition,
encrypted input, Unicode paths, overwrite protection, and error cleanup.

The GUI comparison identified real differences, not assumed equivalence:
Canvas uses one-sided variance, SVG texture, and CSS sepia; Magica and this CLI
use symmetric variance, while the CLI's cream blend follows Magica's tint idea.
The Magica defaults for blur/noise are 0.5/0.25; CLI `gui` uses Canvas's 0.3/0.1.

## Real 314-page PDF

The local Everything index found `ANSYS_Mechanical_APDL_Basic_Analysis_Guide.pdf`,
a software manual under Program Files. Its contents are not distributed here.
The **packaged EXE** processed the entire original, without reducing quality or
selecting a subset, using `--preset subtle --seed 42 --json`.

| Measurement | Observed result on this Windows x64 host |
|---|---|
| Pages | 314 input / 314 output |
| Input | 5,129,044 bytes |
| Output | 182,590,162 bytes (about 35.6 times larger) |
| Settings | Gray, rotate 0 +/-0.3 degrees, blur 0.2, noise 0.08, scale 2 / 144 DPI, JPEG quality 92 |
| Time | 95.907 seconds CLI elapsed; 96.358 seconds measured wall time |
| Peak working set | 691,433,472 bytes / 659.4 MiB |
| Peak private bytes | 1,478,578,176 bytes / 1,410.1 MiB |
| Outcome | Exit 0; JSON `ok: true`; no intervention or workaround |

All 314 page sizes matched 612 x 792 points. Every page contained one JPEG;
all 314 embedded images matched deterministic regeneration from the original
pages/settings/seed, verifying content and order. Rendered pages 1, 158, and 314
were visually inspected and were readable/intact; the last is intentionally
mostly blank. The original file's SHA-256 remained unchanged. The generated PDF,
preview images, and unique temporary directory were removed.

## Practical limits

This is evidence for one representative long document, not unlimited scale.
The CLI renders sequentially but retains compressed page data until PDF saving,
so memory grows with document content. Per-page rasterization is capped at
20 million pixels. Higher resolution, noise, image-heavy pages, or more pages
can exhaust memory and produce large output. The run stayed below the test's
8-minute and roughly 1.5-GiB working-set stop bounds.

The original browser GUI was **not** long-document tested. Its export code starts
all page renders using `Promise.all` and retains the resulting blobs; this is a
code-derived memory-pressure concern, not a measured browser failure.

Runtime identity used for these checks:

- CLI source SHA-256: `db8de9e5c7cfbf065a1e744a353f3317e33e763b805b852c2ced2dceb43bbc0e`
- Packaged EXE SHA-256: `a7d9d34b75608c230d0e317e45c4250b1aae8981edc7f97262504d3ecb5f271c`
