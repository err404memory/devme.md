# devme — Getting Started

This manual walks you through installing and using devme for the first time.

---

## What is devme?

devme gives every directory on your filesystem a companion markdown file — a context note that travels with that location. Your companion files link to each other by directory relationship (parent, siblings, children), forming a navigable mesh you can browse from a local web interface.

The annotation layer is the core feature. You can attach a sticky note to any file or directory without modifying it. Notes are stored separately from your files, rendered inline in the viewer, and keyed to the path — so they persist across sessions.

The section schema in each companion file — Status, Next Steps, Change Log, Session Log — also maps naturally to how AI assistants track project state. Your context mesh and your AI assistant stay in sync without extra effort.

---

## Requirements

- Python 3.10 or newer
- A modern browser (Chrome, Firefox, Safari, Edge)

---

## Installation

**1. Copy the CLI to your PATH:**

```sh
cp ash ~/.local/bin/ash
chmod +x ~/.local/bin/ash
```

Verify it works:

```sh
ash --help
```

**2. Create your hub directory and copy the web interface:**

```sh
mkdir -p ~/.ash
cp serve.html ~/.ash/serve.html
```

**3. Create your config file:**

```sh
cp config.example.json ~/.ash/config.json
```

Then open `~/.ash/config.json` in your editor and fill in your values. At minimum, set `username` and `timezone`. Everything else has sensible defaults.

---

## Configuration

The config file lives at `~/.ash/config.json`. The CLI reads it on every run, falling back to defaults for any missing key.

### Minimal config

```json
{
  "username": "alex",
  "timezone": "America/Chicago"
}
```

### Full config reference

| Key | Default | Description |
|-----|---------|-------------|
| `username` | `"ash"` | Displayed in the sidebar header |
| `filename` | `"me.md"` | Companion filename created in each directory |
| `hub_label` | `"My Context Hub"` | Browser tab and sidebar title |
| `hub_dir` | `"~/.ash"` | Where your global companion file and `serve.html` live |
| `editor` | `"code"` | Editor launched by the "Open in …" button in the interface |
| `accent_color` | `"#7b96e8"` | Primary accent color throughout the UI |
| `timezone` | `"UTC"` | Timezone used for timestamps in companion files |
| `notes_file` | `~/.ash/file-notes.json` | Where annotations are stored (leave blank for default) |
| `server_prefix_local` | `""` | Local mount path for a remote filesystem — see Multi-device below |
| `server_prefix_remote` | `""` | Corresponding path on the remote — see Multi-device below |

**`filename`** — this is the companion filename devme creates and looks for in every directory. Set it to your name, your initials, or anything consistent. That filename becomes your file everywhere. Example: `"alex.md"` or `"me.md"`.

**`editor`** — the command name or path that opens files in your editor. Common values: `"code"` (VS Code), `"zed"`, `"nvim"`, `"subl"`.

**`accent_color`** — any CSS hex color. The default blue-violet works well on both light and dark backgrounds.

---

## First Run

**Start the interface:**

```sh
ash serve
```

Open the URL shown in the terminal (default: `http://localhost:7272`). You'll see an empty sidebar — no projects yet.

**Initialize your first companion file:**

```sh
ash init ~/projects/my-project
```

This creates `~/projects/my-project/me.md` (or whatever your `filename` is set to), registers it in your global index, and back-propagates links to any neighboring companion files.

Reload the interface — your project appears in the sidebar. Click it to see the companion file rendered with full markdown support.

---

## Daily Workflow

```sh
ash serve                    # open the interface in your browser
ash init [path]              # create a companion file in a new directory
ash update [path]            # refresh navigation links in an existing companion file
ash refresh                  # rebuild navigation in all registered companion files
ash push [path]              # write Status/Next Steps back to a project overview file
ash watch [path]             # keep companion file and project overview in sync on save
ash upgrade [--all]          # migrate older companion files to the current format
ash rename-overview <name>   # rename project overview files across all registered projects
```

**Typical session:**
1. `ash serve` — open the interface
2. Browse to the project you're working on — check Status, Next Steps, Session Log
3. Do your work
4. Append a note to Session Log in the companion file
5. `ash update [path]` if you changed the project overview

The interface live-reloads whenever a companion file changes, so you can keep it open alongside your editor.

---

## Annotations

Annotations are notes you attach to files and directories without modifying them.

In the interface, click the **✦** icon next to any item in the directory tree to add or edit a note. The note for the current page renders as a styled box below the title.

Notes are stored in `~/.ash/file-notes.json` (configurable via `notes_file`). They are never written into your repos or folders.

---

## Companion File Format

`ash init` creates a structured markdown file with a fixed section schema:

```markdown
# project-name
`/path/to/project`

## Navigation
- [My Context Hub](~/.ash/me.md)
- Up: [parent-dir](/path/to/parent/me.md)
- Nearby: [sibling-project](/path/to/sibling/me.md)
- Down: [child-project](/path/to/child/me.md)

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
```

**Sections maintained automatically by the CLI:** Navigation, Directory, AI Notes, Change Log.

**Sections that are yours:** Status, Next Steps, Session Log — write freely, or sync them from a project overview file.

**AI Notes** — automatically detects `CLAUDE.md`, session summaries, and other AI-generated context in the directory and links to them.

---

## whatdoing Integration

If you use [`whatdoing`](https://github.com/err404memory/whatdoing) for project management, devme can sync with its overview files (`project.md` or `_OVERVIEW.md`).

When a project overview exists in a directory, `ash init` automatically pulls **Status** and **Next Steps** into the companion file. After that:

```sh
ash update [path]   # pull latest status from project.md into companion file
ash push [path]     # write your edits back to project.md
ash watch [path]    # two-way sync on file save
```

The project overview stays the source of truth. The companion file surfaces what matters right now.

---

## Multi-device Setup

If your filesystem is mounted across machines at different paths (e.g. a server mounted via rclone or SSHFS), set `server_prefix_local` and `server_prefix_remote` so annotations resolve correctly on each machine.

**Example:** You have a server whose `/home/user/` is mounted on your laptop at `~/server/`.

```json
{
  "server_prefix_local":  "~/server",
  "server_prefix_remote": "/home/user"
}
```

Annotations are keyed to a device-agnostic path internally. The same note is visible whether you're on the server directly or accessing its files through the mount on your laptop.

---

## Troubleshooting

**`ash: command not found`**
Make sure `~/.local/bin` is on your PATH. Add this to your shell config (`~/.bashrc` or `~/.zshrc`) and restart your terminal:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

**Interface loads but sidebar is empty**
Run `ash init [path]` on at least one directory first. The sidebar only shows registered projects.

**Companion file exists but doesn't appear in sidebar**
Run `ash refresh` to rebuild the global index. This picks up any files that were created manually or outside the CLI.

**Annotations not saving**
Check that `~/.ash/` is writable and that `notes_file` (if customized) points to a valid path.

**Timestamps are wrong**
Set `timezone` in your config to your local timezone. Use IANA format, e.g. `"America/New_York"`, `"Europe/London"`, `"Asia/Tokyo"`. A list is available at [Wikipedia: List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

**`ash serve` port already in use**
Another process is using port 7272. You can kill it or temporarily change the port with `ash serve --port 7273`.

---

## License

MIT
