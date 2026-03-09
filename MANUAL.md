# devme — Getting Started

This manual walks you through installing and using devme for the first time.

---

> **Note:** Throughout this documentation, `me.md` is used as the companion filename placeholder.
> After running `devme install`, your actual configured filename (e.g. `alex.md`) will be used on your system.
> Wherever docs say `me.md`, substitute your configured filename.

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

**1. Install devme:**

```sh
pipx install devme-md
```

PyPI package name is `devme-md`; the installed command is `devme`.

> **pipx note:** If `pipx` is missing, install it first (`python3 -m pip install --user pipx`) and run `python3 -m pipx ensurepath`.

Verify it works:

```sh
devme --help
```

From a local clone instead of PyPI:

```sh
pipx install .
```

**2. Run the setup wizard:**

```sh
devme install
```

A browser window opens with a step-by-step setup wizard. It walks you through:

- Choosing your companion filename (e.g. `alex.md` or `me.md`)
- Setting your name, sidebar title, editor, timezone, and accent color
- A live preview of the interface accent color as you pick it

On finish, the wizard:
- Creates `~/.devme/config.json` with your values
- Installs bundled UI assets (`serve.html`, `wizard.html`) into `~/.devme/`
- Creates your global hub file (`~/.devme/<your-filename>`)
- Writes `~/.devme/QUICKSTART.md` — a personalized quick reference

To reconfigure, run `devme install --force` or edit `~/.devme/config.json` directly.

---

## Configuration

The config file lives at `~/.devme/config.json`. The CLI reads it on every run, falling back to defaults for any missing key.

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
| `username` | `"you"` | Displayed in the sidebar header |
| `filename` | `"me.md"` | Companion filename created in each directory |
| `hub_label` | `"My Context Hub"` | Browser tab and sidebar title |
| `hub_dir` | `"~/.devme"` | Where your global companion file and `serve.html` live |
| `editor` | `"code"` | Editor launched by the "Open in …" button in the interface |
| `accent_color` | `"#7b96e8"` | Primary accent color throughout the UI |
| `timezone` | `"UTC"` | Timezone used for timestamps in companion files |
| `notes_file` | `~/.devme/file-notes.json` | Where annotations are stored (leave blank for default) |
| `server_prefix_local` | `""` | Local mount path for a remote filesystem — see Multi-device below |
| `server_prefix_remote` | `""` | Corresponding path on the remote — see Multi-device below |

**`filename`** — this is the companion filename devme creates and looks for in every directory. Set it to your name, your initials, or anything consistent. That filename becomes your file everywhere. Example: `"alex.md"` or `"me.md"`.

**`editor`** — the command name or path that opens files in your editor. Common values: `"code"` (VS Code), `"zed"`, `"nvim"`, `"subl"`.

**`accent_color`** — any CSS hex color. The default blue-violet works well on both light and dark backgrounds.

---

## First Run

1. Start the interface:

```sh
devme serve
```

Open the URL shown in the terminal (default: `http://localhost:7272`). On first load, the guided tour starts automatically.

2. Guided tour behavior:

- If contexts already exist, the tour walks the sidebar, content area, annotations, and sync loop.
- If no contexts exist yet, the tour points you to the `devme init` command to run next.

3. Initialize your first companion file:

```sh
devme init ~/projects/my-project
```

This creates `~/projects/my-project/me.md` (or whatever your `filename` is set to), registers it in your global index, and links it with neighboring companion files.

4. Reload the interface. Your project appears in the sidebar, and you can replay the walkthrough any time with the top-bar `Tour` button.

---

## Daily Workflow

```sh
devme serve                    # open the interface in your browser
devme init [path]              # create a companion file in a new directory
devme update [path]            # refresh navigation links in an existing companion file
devme refresh                  # rebuild navigation in all registered companion files
devme push [path]              # write Status/Next Steps back to a project overview file
devme watch [path]             # keep companion file and project overview in sync on save
devme upgrade [--all]          # migrate older companion files to the current format
devme rename-overview <name>   # rename project overview files across all registered projects
```

**Typical session:**
1. `devme serve` — open the interface
2. Browse to the project you're working on — check Status, Next Steps, Session Log
3. Do your work
4. Append a note to Session Log in the companion file
5. `devme update [path]` if you changed the project overview

The interface live-reloads whenever a companion file changes, so you can keep it open alongside your editor.

---

## Annotations

Annotations are notes you attach to files and directories without modifying them.

In the interface, click the **✦** icon next to any item in the directory tree to add or edit a note. The note for the current page renders as a styled box below the title.

Notes are stored in `~/.devme/file-notes.json` (configurable via `notes_file`). They are never written into your repos or folders.

---

## Companion File Format

`devme init` creates a structured markdown file with a fixed section schema:

```markdown
# project-name
`/path/to/project`

## Navigation
- [My Context Hub](~/.devme/me.md)
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
| 2026-03-08 14:30 | Created by `devme init` |

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

If you use [`whatdoing`](https://github.com/err404memory/whatdoing) for project management, devme can sync with its overview files. Canonical filename is `overview.md`; legacy names are still read: `project.md`, `_OVERVIEW.md`, `PROJECT.md`, and `devme.md`.

When a project overview exists in a directory, `devme init` automatically pulls **Status** and **Next Steps** into the companion file. After that:

```sh
devme update [path]   # pull latest status from overview file into companion file
devme push [path]     # write your edits back to overview file
devme watch [path]    # two-way sync on file save
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

## Session Auto-Logging

The `hooks/` directory provides an optional pipeline that scans your local AI tool logs and terminal session logs — whatever exists on your system — and appends context to your companion files automatically.

devme has no AI dependency. The hooks are log parsers. `session-close` checks known locations for recently-closed sessions from Claude Code, Kilo, Codex, Aider, Ghostty, tmux, or any tool you configure. It reads those logs and extracts what was discussed — decisions, ideas, generated code, session summaries — then writes that context to the Session Log of the companion file for the relevant project directory. No external service is contacted. Nothing is sent anywhere.

### How it works

On terminal exit, `session-close` checks all configured AI tool log directories for recently-closed sessions. For each one it finds, it dispatches to `parse-ai-session` (JSONL-format logs: Claude Code, Kilo, Codex, Aider, and others) or `parse-ghostty-session` (text/terminal logs). The resulting summary is passed to `update-session-docs`, which appends it to the `## Session Log` section of the companion file for whatever project directory was active.

You can also call `update-session-docs` directly to append a manual note without any log file:

```sh
update-session-docs --summary "Fixed the auth bug, pushed to staging" \
  --cwd ~/projects/my-app --tool manual --date "$(date -Iseconds)"
```

### Setup

See the **Session auto-logging** section of the README for full installation steps.

### Session log format

Each entry looks like:

```markdown
---

### 2026-03-08 14:32 | claude

## Key Points
- Implemented the auth flow
- Resolved the CORS issue on the staging endpoint

## Takeaway
Auth flow complete and tested. Staging deploy blocked on SSL cert renewal.
```

Entries stack newest-last inside the `## Session Log` section and are rendered as collapsible entries in the devme viewer.

---

## Troubleshooting

**`devme: command not found`**
Make sure `~/.local/bin` is on your PATH. Add this to your shell config (`~/.bashrc` or `~/.zshrc`) and restart your terminal:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

**Migrating from early preview builds (`ash`)**
Run `devme install --force` once. This refreshes bundled UI assets and writes config into `~/.devme/`. Legacy `~/.ash/config.json` is still read during transition.

**Interface loads but sidebar is empty**
Run `devme init [path]` on at least one directory first. The sidebar only shows registered projects.

**Guided tour doesn't appear**
The tour runs automatically only the first time in each browser profile. Click the `Tour` button in the top bar to replay it any time.

**Companion file exists but doesn't appear in sidebar**
Run `devme refresh` to rebuild the global index. This picks up any files that were created manually or outside the CLI.

**Annotations not saving**
Check that `~/.devme/` is writable and that `notes_file` (if customized) points to a valid path.

**Timestamps are wrong**
Set `timezone` in your config to your local timezone. Use IANA format, e.g. `"America/New_York"`, `"Europe/London"`, `"Asia/Tokyo"`. A list is available at [Wikipedia: List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

**`devme serve` port already in use**
Another process is using port 7272. You can kill it or temporarily change the port with `devme serve --port 7273`.

---

## License

MIT
