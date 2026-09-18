# worky

WSL helper script that opens tab-colored, auto-tiled Windows Terminal windows
for project directories, with an optional command typed into each on launch.

## Key facts

- Single-file bash script: `worky` (currently v1.0.8, version string at line 6).
- Docs: `README.md` (usage, flags, examples).
- Windows-only functionality: drives `wt.exe` / `powershell.exe` from WSL, so it
  cannot run or be tested on plain Linux/macOS.
- Runtime state: `.worky-color-state` (tab-color rotation) and `.worky-tmp`
  (generated PowerShell scripts), both created next to the script itself.
- No test suite, no CI, no package manifest - this is a personal utility script.

## Status

- Initial commit + README added. No open issues tracked yet.

## Working notes

- Since this can't be exercised on this (Linux) machine, verify changes by
  reading the script and reasoning through the PowerShell/bash logic rather
  than by running it end-to-end.
