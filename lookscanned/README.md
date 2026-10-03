# LookScanned portable CLI and skill

Windows x64, offline PDF scan effects with a bundled runtime. No Python, Node,
Electron, or application source checkout is needed to run it. Keep this folder
together, including `bin/_internal`.

From this folder:

```powershell
.\bin\lookscanned.exe "input.pdf" -o "input-scan.pdf" --preset subtle --json --quiet
```

See [all options, defaults, ranges, and parameter behavior](references/options.md).
The bare CLI defaults to `gui`; the agent skill normally chooses `subtle`.
Visible content is rasterized at the original page sizes. Searchable text,
links, interactive forms, and bookmarks are not retained. The original file is
preserved. Effects are not pixel-identical to the browser GUI.

To install in Codex, copy this complete `lookscanned` folder into your skills
directory, normally `%USERPROFILE%\.codex\skills`, or ask Codex to install from:
[billerss11/lookscanned-skill](https://github.com/billerss11/lookscanned-skill/tree/main/lookscanned).
The skill name is `lookscanned`. Existing installations do not auto-update.

Runtime versions are in `BUILD-INFO.json`; dependency licenses are under
`THIRD_PARTY_LICENSES` and listed in `THIRD_PARTY_NOTICES.txt`.
