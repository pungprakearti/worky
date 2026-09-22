# worky

A WSL (Windows Subsystem for Linux) helper script that opens Windows Terminal
windows for your project directories, tab-colored and auto-tiled to fill your
screen, with an optional command typed into each one on launch.

Two modes:

- **Picker mode** (no flags, or just `-c`): shows an interactive grid of the
  non-hidden subdirectories in your current working directory (up to 4
  columns, adapting to your terminal width), lets you select up to 4 of them,
  then opens one Windows Terminal window per selection and snaps them into
  place (maximize / halves / thirds / quadrants depending on how many you
  picked).
- **Direct mode** (`-d PATH`): skips the picker and opens a single terminal
  for the given path.

Each opened window gets a `--tabColor` from a small rotating palette, and the
window title is set to the directory's name. In both modes, every window also
gets a second tab in the *same* window, pointed at the same folder, titled
"`<dirname> directory`" and left at the terminal's default color; focus
returns to the first tab once the second one opens.

## Requirements

- WSL running Ubuntu, with Windows Terminal (`wt.exe`) and `powershell.exe`
  reachable from the WSL shell (this is the default on Windows 11 / recent
  Windows 10 WSL setups).
- `wslpath` (ships with WSL).
- Bash 4+ (for associative-array-free features used, e.g. `mapfile`).

The script drives Windows Terminal and window placement via small inline
PowerShell/C# snippets (Win32 `user32.dll` calls), so it only works from
inside WSL on Windows, not on plain Linux/macOS.

## Installation

Put `worky` somewhere on your `PATH` (or symlink it there) and make sure it's
executable:

```bash
chmod +x worky
ln -s "$(pwd)/worky" ~/bin/worky   # or wherever your PATH looks
```

The script tracks its tab-color rotation in a `.worky-color-state` file, and
uses a `.worky-tmp` directory for generated PowerShell scripts, both created
next to the `worky` script itself.

## Usage

```
worky [-c COMMAND] [-d PATH]
```

### Flags

| Flag | Long form | Description |
|------|-----------|-------------|
| `-c COMMAND` | `--command COMMAND` | Type `COMMAND` into each opened terminal and press Enter, once the shell inside it is ready. Works in both picker mode and direct mode. |
| `-d PATH` | `--dir PATH` | Open a single terminal for `PATH` directly, skipping the interactive picker. The tab color still advances through the normal rotation. |
| `-v` | `--version` | Print the script's version and exit. |
| `-h` | `--help` | Print usage help and exit. |

Notes:

- `PROJECTS_DIR` for the picker is always your current working directory
  (`pwd`) when you run `worky` -- run it from the folder containing the
  project directories you want to choose from.

## Picker mode (default)

Running `worky` with no `-d` flag lists every immediate, non-hidden
subdirectory of the current directory (directories starting with `.` are
skipped) in a grid of up to 4 columns:

```bash
cd ~/projects
worky
```

Controls inside the picker:

| Key | Action |
|-----|--------|
| Up / Down / Left / Right | Move the cursor around the grid |
| Space | Select or deselect the highlighted directory (up to 4) |
| Enter | Launch terminals for everything selected |
| q / Q | Cancel and exit without launching anything |

Selected entries are numbered in the order you picked them (`[1]`, `[2]`,
...) -- this order determines the on-screen tiling position:

| Count selected | Layout |
|-----------------|--------|
| 1 | Maximized |
| 2 | Left half / right half |
| 3 | Left-top, left-bottom, right (large) |
| 4 | Four quadrants (top-left, top-right, bottom-left, bottom-right) |

All windows are placed on whichever monitor your current terminal window is
on when you invoke `worky`. Each window also gets a second "directory" tab
(see above), and focus returns to the first tab afterward.

### Examples

Just browse and open, no command typed in:

```bash
cd ~/projects
worky
```

Open your picks and immediately run `npm run dev` in each of them:

```bash
cd ~/projects
worky -c "npm run dev"
```

## Direct mode (`-d`)

Skips the picker entirely and opens exactly one terminal for the given path:

```bash
worky -d ~/projects/my-app
```

Open it and immediately run a command:

```bash
worky -d ~/projects/my-app -c "git status"
```

As in picker mode, a second "directory" tab is opened alongside the first
(e.g. one tab for running a dev server, one tab left as a plain shell in the
same directory), and focus returns to the first tab. Combined with `-c`, the
command only runs in the first tab; the second tab (`<dirname> directory`) is
opened as a plain shell:

```bash
worky -d ~/projects/my-app -c "npm run dev"
```

## How it works (brief)

- Picker mode enumerates directories with `find "$PROJECTS_DIR" -mindepth 1 -maxdepth 1 -type d -not -name '.*'`.
- For each selected/direct target, `worky` launches `wt.exe` with `-p Ubuntu`,
  a `--title`, and a `--tabColor`, pointed at the Windows path for that
  directory (converted via `wslpath -w`).
- When a command (`-c`) is given, or multiple windows need positioning, the
  script generates a temporary PowerShell script (under `.worky-tmp`) that
  uses Win32 API calls to find the newly opened window by title, move/snap
  it into place, and type the command into it via synthesized keystrokes.
- After the first tab is set up, `worky` opens a second "directory" tab via
  `wt.exe new-tab`, then chains `focus-tab -t 0` in the same invocation to
  return focus to the first tab.
- Temporary PowerShell scripts are deleted after each run.

## Known limitations

- Windows/WSL + Windows Terminal only; there's no fallback for other
  terminals or OSes.
- Window-finding matches by exact title, so two directories with the same
  basename selected/opened in the same run could be positioned unreliably.
- Snap placement relies on Windows' Snap Layouts keyboard shortcuts
  (`Win+Arrow`), so it depends on Snap being enabled in Windows settings.
