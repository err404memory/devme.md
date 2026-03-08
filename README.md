# devme

Context notes for your filesystem. Every directory gets a companion markdown file — attach notes to any file or folder, navigate your whole codebase from one local interface, and keep project status in sync with your project manager without touching anything it annotates.

---

## The idea

A well-organized filesystem is still opaque without context. What is this directory for? Why does this file exist? What was in progress when you last had this project open?

devme gives every directory a companion file (`you.md`, named for you — configurable). That file holds whatever context belongs to that location: status, next steps, a live directory tree, a change log, a session log. Each file links to its neighbors — parent, siblings, children — forming a navigable mesh across your whole filesystem.

The annotation layer is the core feature. You can attach a note to any file or directory in the interface without modifying it. The note is stored separately from your files, appears inline when you open that location in the viewer, and travels with the path across sessions. It's the equivalent of a sticky note on a manila folder in a filing cabinet — the folder's contents are unchanged, but the context is there every time you pull it out: what it's for, what's in progress, what to watch out for.

That same structure makes the files easy for AI assistants to read and maintain alongside you. The consistent section schema — Status, Next Steps, Change Log, Session Log — maps directly to how an AI assistant tracks project state, so your context mesh and your AI context stay naturally in sync.

<!-- screenshot: full interface — sidebar project tree on the left, rendered context file on the right showing status card and navigation links -->

---

## Structure

```
ash          CLI — init, serve, sync, manage
serve.html   Web interface (single-file SPA, loaded from ~/.ash/)
hooks/       Session auto-logging pipeline (optional, see below)
```

---

## Setup

**Requirements:** Python 3.10+, a modern browser.

```sh
# Place the CLI on your PATH
cp ash ~/.local/bin/ash
chmod +x ~/.local/bin/ash

# Copy the web interface
mkdir -p ~/.ash
cp serve.html ~/.ash/serve.html
```

Create `~/.ash/config.json`:

```json
{
  "username": "you",
  "filename": "you.md",
  "hub_label": "My Mesh",
  "hub_dir": "~/.ash",
  "editor": "zed",
  "accent_color": "#7b96e8",
  "timezone": "America/New_York"
}
```

`filename` is the companion file devme creates and looks for in each directory. Set it to your name, your initials, or anything consistent — that filename becomes your file everywhere.

---

## Usage

```sh
ash serve                    # start the local interface (default: localhost:7272)
ash init [path]              # create a companion file in a directory
ash update [path]            # refresh navigation links + pull status from project overview
ash push [path]              # write status changes back to the project overview file
ash watch [path]             # keep companion file and project overview in sync on save
ash refresh                  # rebuild navigation in all registered companion files
ash upgrade [--all]          # migrate older companion files to the current format
ash rename-overview <name>   # rename project overview files across registered projects
ash rm [path]                # remove a directory from the mesh index
ash rm [path] --delete       # remove from index and also delete the companion file
```

`ash init` registers the new file in your global index and back-propagates: neighboring companion files in parent and sibling directories automatically gain links to the new one.

---

## Interface

<!-- screenshot: close-up of the sidebar tree showing directory hierarchy with A-Z toggle and bookmarks panel -->

**Sidebar** — registered projects as a navigable directory tree, grouped by path. Drag the right edge to resize. Toggle between tree view and A–Z sort. Bookmarks panel at the bottom.

**Content area** — renders the companion file with full markdown support: tables with sortable columns, fenced code with syntax highlighting, collapsible sections. The Directory section renders as a live clickable file tree — every file and subfolder is a link you can open directly in the viewer or in your editor.

<!-- screenshot: annotation box below the page title — ✦ icon in header, note rendered as markdown below the filepath bar, ghost "✦ Add context" state visible on a folder in the directory tree -->

**Annotations** — the core feature. Attach a context note to any file or directory without modifying it. Click ✦ next to any item in the directory tree to add or edit a note. The note for the currently open page renders as a styled box below the title. Notes are stored in a separate JSON file and never written into your repos or folders.

<!-- screenshot: navigation section showing Up (purple), Nearby (amber), Down (green) color-coded links between project directories -->

**Navigation** — companion files link to each other by directory relationship. Parent directories appear in purple (↑), siblings in amber (↔), children in green (↓). The mesh self-assembles as you run `ash init` in new locations.

**Live reload** — an SSE connection watches all registered files and reloads the viewer automatically when any of them change.

---

## whatdoing sync

devme integrates with [`whatdoing`](https://github.com/err404memory/whatdoing) project overview files. These are the detailed per-project documents maintained by whatdoing — tech stack, commands, status, roadmap, blockers — the full project record. The companion file is intentionally lighter: navigation, annotations, session log, and a live window into the overview's current status.

When a `project.md` or `_OVERVIEW.md` exists in a directory, `ash init` pulls its **Status** and **Next Steps** fields into the companion file automatically. From that point you can keep them in sync:

```sh
ash update [path]   # pull latest from project.md into companion file
ash push [path]     # write Status/Next Steps changes back to project.md
ash watch [path]    # continuous two-way sync on file save
```

<!-- screenshot: Status card in the viewer showing project status pulled from whatdoing's project.md — Status and Next Steps rendered as a single bordered card -->

The project overview remains the source of truth. The companion file surfaces what matters now — without duplicating the documentation.

---

## Session auto-logging

The `hooks/` directory contains scripts that close the loop between your terminal sessions and your companion files. On every terminal exit, your session is summarized by an AI and appended to the relevant companion file's Session Log automatically — no manual notes required.

### How it works

```
terminal exit
  └─ session-close              (bash EXIT trap)
       ├─ parse-ai-session      (Claude Code JSONL → summary via claude -p)
       └─ parse-ghostty-session (terminal log → summary via claude -p)
            └─ update-session-docs  (appends summary to companion files)
```

The session summary is structured (Key Points, Decisions, Takeaway) and written into the `## Session Log` section of both the global hub and the project companion file for whatever directory you were working in.

### Install

```sh
cp hooks/session-close         ~/.local/bin/
cp hooks/update-session-docs   ~/.local/bin/
cp hooks/parse-ai-session      ~/.local/bin/
cp hooks/parse-ghostty-session ~/.local/bin/
cp hooks/tmux-start-log        ~/.local/bin/
chmod +x ~/.local/bin/session-close \
         ~/.local/bin/update-session-docs \
         ~/.local/bin/parse-ai-session \
         ~/.local/bin/parse-ghostty-session \
         ~/.local/bin/tmux-start-log
```

Add to `~/.bashrc` (inside the interactive shell guard):

```sh
# devme session logging
trap '~/.local/bin/session-close' EXIT
```

For tmux, add to `~/.tmux.conf`:

```
set-hook -g after-new-window   'run "~/.local/bin/tmux-start-log #{session_name} #{window_index} #{pane_id}"'
set-hook -g after-split-window 'run "~/.local/bin/tmux-start-log #{session_name} #{window_index} #{pane_id}"'
```

### Config

`vault_dir` sets where full structured session archives are saved (organized by `YYYY-MM/`). Set it to a path on your server mount for cross-device access. The companion file Session Log gets a concise summary; the vault gets the full structured record.

The session detection threshold defaults to 300 seconds. Override with:

```sh
export ASH_SESSION_THRESHOLD=600   # 10 minutes
```

### Dolphin integration

`hooks/ash-init` and `hooks/ash-open` are service menu wrappers. They fork immediately so Dolphin doesn't freeze, use `notify-send` for desktop feedback, and resolve the `ash` binary from `$PATH`.

```sh
cp hooks/ash-init  ~/.local/bin/
cp hooks/ash-open  ~/.local/bin/
chmod +x ~/.local/bin/ash-init ~/.local/bin/ash-open
```

---

## Companion file format

`ash init` generates a structured file with a fixed section schema:

````markdown
# project-name

## Navigation
- [My Mesh](~/.ash/you.md)
- Up: [parent-dir](/path/to/parent/you.md)
- Nearby: [sibling-project](/path/to/sibling/you.md)

> [README](./README.md)

---

## Status

_No status set._

## Next Steps

_No next steps defined._

---

## Change Log

| Date | Event |
|------|-------|
| 2026-03-08 14:30 | Created by `ash init` |

---

## Directory

project-name/
- [README.md](./README.md)
- **src/**
  - [main.py](./src/main.py)

---

## AI Notes

<!-- No AI notes found -->

---

## Session Log

<!-- Auto-appended by session-close on terminal exit -->
````

**Navigation**, **Directory**, **AI Notes**, and **Change Log** are maintained automatically by the CLI. **Status**, **Next Steps**, and **Session Log** are yours — or synced from your project overview.

The AI Notes section automatically detects `CLAUDE.md`, session summary files, and other AI-generated context in the directory and links them here.

---

## Config reference

| Key | Default | Description |
|-----|---------|-------------|
| `username` | `"ash"` | Displayed in the sidebar header |
| `filename` | `"ash.md"` | Companion filename looked for in each directory |
| `hub_label` | `"Context Mesh"` | Browser tab and sidebar title |
| `hub_dir` | `"~/.ash"` | Location of the global companion file and `serve.html` |
| `editor` | `"code"` | Editor launched by the "Open in …" button |
| `accent_color` | `"#7b96e8"` | Primary accent color throughout the UI |
| `timezone` | `"UTC"` | Timezone for timestamps |
| `notes_file` | `~/.ash/file-notes.json` | Where annotations are stored |
| `vault_dir` | `~/Documents/sessions` | Where full session archives are saved by the hooks |
| `server_prefix_local` | `""` | Local mount path for a remote filesystem (e.g. `~/server`) |
| `server_prefix_remote` | `""` | Corresponding path on the remote machine (e.g. `/home/user`) |
| `icon_folder` | built-in SVG | Closed folder icon — any SVG `url()` or data URI |
| `icon_folder_open` | built-in SVG | Open folder icon |
| `ann_icon_color` | accent color | Color of the ✦ annotation icon |

### Multi-device paths

If your filesystem is mounted across machines at different paths (e.g. a server mounted via rclone), set `server_prefix_local` and `server_prefix_remote`. Annotations are keyed to a device-agnostic path and resolve correctly on each machine — the same note is visible whether you're on the machine that owns the files or accessing them through a mount.

---

## Roadmap

- **Dolphin panel** — embed the viewer as a native preview pane in KDE Dolphin; context notes appear as you browse without opening a browser
- **Unified filesystem view** — single interface across local directories, mounted remotes, and SSH paths
- **File manager plugins** — layered integration with Nautilus, Thunar, and other managers as an alternative to the standalone server

---

## License

MIT
