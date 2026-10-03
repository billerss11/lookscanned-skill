# LookScanned skill for Windows

A ready-to-install Codex skill that makes PDFs look scanned. The complete
Windows x64 command-line application is bundled in `lookscanned/bin`.
No Python, Node.js, Electron, compiler, or application source checkout is needed
to run it. Processing is local and offline after installation.

## Install in Codex

Ask Codex:

> Install the lookscanned skill from https://github.com/billerss11/lookscanned-skill/tree/main/lookscanned

The installable skill is the **`lookscanned/` subfolder**, not the repository root.
The skill name is **`lookscanned`**. Use it by saying, for example:

> Use $lookscanned to make this PDF look lightly scanned, preserve color, and save a new copy.

For manual installation, download this repository's ZIP and copy its complete
`lookscanned` folder into your Codex skills directory, normally
`%USERPROFILE%\.codex\skills`. Keep the whole `bin` directory with the skill.
Do not copy only `SKILL.md` or only the EXE. Private GitHub credentials are not
required for this public repository.

## Run directly

From this repository's root:

```powershell
.\lookscanned\bin\lookscanned.exe "input.pdf" -o "input-scan.pdf" --preset subtle --seed 42
.\lookscanned\bin\lookscanned.exe --help
```

All ten original GUI controls are available: colorspace, border, rotation,
rotation variance, brightness, paper tint, contrast, blur, noise, and resolution.
Presets, selected pages, PNG/JPEG page encoding, reproducible seeds, and JSON
results are also supported. See [full CLI usage](lookscanned/README.md).

The original PDF is preserved. Output contains rasterized pages at the original
physical sizes, so editable text, links, and forms are not retained. This is a
native implementation of the scan controls; its appearance is not pixel-identical
to the original browser application. Current binaries target Windows x64 only.

## Manage and update

This repository tracks the complete installable skill, including runtime files.
Update instructions in `lookscanned/SKILL.md`. When updating the application,
replace the complete `lookscanned/bin` directory from a tested portable build,
along with its build information and third-party notices, then commit and push.
Do not mix runtime files from different versions.

Existing installations do not update automatically. Keep any local customization
before replacing the installed skill with a newer complete folder. If you manage
a separate clone of this repository, pull updates there and copy its `lookscanned`
folder into your skills directory.

The Git history includes executable releases, so cloning history can grow over
time. Downloading a repository ZIP fetches just the selected version. There are
no Git LFS pointers or submodules: the downloaded folder contains real binaries.

## Credits and licenses

Based on [Look Scanned Community Edition](https://github.com/rwv/lookscanned.io).
The project license is retained in [LICENSE](LICENSE). Bundled dependencies have
their own licenses; see [third-party notices](lookscanned/THIRD_PARTY_NOTICES.txt)
and `lookscanned/THIRD_PARTY_LICENSES/`. Build versions are recorded in
[BUILD-INFO.json](lookscanned/BUILD-INFO.json).
