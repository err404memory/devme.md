# devme

Context notes for your filesystem. Each tracked directory gets a companion Markdown file so you can keep status, notes, next steps, and annotations attached to where the work actually lives without modifying the underlying files.

---

> **Note:** Throughout this documentation, `me.md` is used as the companion filename placeholder.
> After running `devme install`, your actual configured filename (e.g. `alex.md`) will be used on your system.
> Wherever docs say `me.md`, substitute your configured filename.

## Uses

- project context that lives with the project folder
- notes attached to files or directories without modifying them
- local status and next steps beside active work
- a cross-project context layer over your filesystem
- a durable “what is this, what changed, what matters here?” trail
- AI-readable project context that stays in plain Markdown

## Concept

`devme` adds one companion Markdown file to each directory you choose to track. That file holds local context for that location: status, next steps, a directory view, a change log, and a session log. Companion files link to parents, siblings, and children, so tracked directories form a navigable mesh.

The annotation layer is the core feature. You can attach a note to any file or directory in the interface without modifying the underlying file. Notes are stored separately, rendered inline in the viewer, and keyed to the path.

The default section schema — Status, Next Steps, Change Log, Session Log — also makes the files easy to maintain with AI assistants if you use them.

## Fit

Best for:

- a context layer over real directories instead of a separate notes silo
- filesystem-local notes that stay attached to where the work happens
- annotations without editing the files you are annotating
- a browser-based view over plain markdown companion files

Less useful for:

- a standalone notes app with no connection to your directories
- a project dashboard first and a filesystem layer second
- a system that hides the markdown files from you

## Examples

`devme` works on its own. It is most useful when your work is already organized by directories and you want context to stay attached to that structure.

Examples:

- a projects folder with many active repos
- a homelab tree with services, scripts, and deployment notes
- a research/workspace directory with long-lived notes beside source files
- a mixed personal/work machine where you need context to survive interruptions

<!-- screenshot: full interface — sidebar project tree on the left, rendered context file on the right showing status card and navigation links -->

---

## Start

Install and run the setup wizard:

```sh
pipx install devme-md
devme install
```

Then create your first companion file and open the interface:

```sh
devme init ~/projects/my-project
devme serve
```

Success criteria:

- the browser opens the local interface
- `~/projects/my-project/me.md` (or your configured filename) exists
- the project appears in the sidebar
- you can click it and read or edit its context

You do not need `whatdoing`, session logging, Dolphin helpers, or any other integration for this first success path.

## Compatibility

- **Tested:** Linux
- **Likely works:** macOS
- **Use WSL for now:** Windows native path handling in the web viewer is still being hardened

Requirements:

- Python 3.10+
- a modern browser

If your system is not the default target:

- on Windows, use WSL for the best current experience
- if you do not want shell hooks, skip the optional `hooks/` setup entirely
- if you do not use `whatdoing`, skip that integration entirely
- if you only want the core tool, you only need `devme install`, `devme init`, and `devme serve`

## Layout

```
devme        CLI — init, serve, sync, manage
serve.html   Web interface (single-file SPA, loaded from ~/.devme/)
wizard.html  Setup wizard (browser-based, opened by devme install)
hooks/       Session auto-logging pipeline (optional, see below)
```

---

## Install

**Requirements:** Python 3.10+, a modern browser.

```sh
pipx install devme-md
devme install
```

PyPI package name is `devme-md`; the installed command is `devme`.

> **pipx note:** If `pipx` is missing, install it first (`python3 -m pip install --user pipx`) and run `python3 -m pipx ensurepath`.

From a local clone (development install):

```sh
pipx install .
devme install
```

`devme install` opens a browser-based setup wizard. It walks you through choosing your companion filename, editor, timezone, and accent color, and shows a live preview of the interface while you do it.

When it finishes, the wizard creates `~/.devme/config.json`, installs bundled UI assets (`serve.html`, `wizard.html`) into `~/.devme/`, creates your global hub file, and writes `~/.devme/QUICKSTART.md`.

After setup, run `devme init` for at least one project directory, then launch `devme serve`. On first load, the interface starts a guided tour automatically. If you have no contexts yet, the tour shows the exact `devme init` command to run next. You can replay the walkthrough any time with the `Tour` button in the top bar.

---

## Commands

```sh
devme install                  # browser-based setup wizard (run once, or devme install --force to redo)
devme serve                    # start the local interface (first load starts the guided tour)
devme init [path]              # create a companion file in a directory
devme update [path]            # refresh navigation links + pull status from project overview
devme push [path]              # write status changes back to the project overview file
devme watch [path]             # keep companion file and project overview in sync on save
devme refresh                  # rebuild navigation in all registered companion files
devme upgrade [--all]          # migrate older companion files to the current format
devme rename-overview <name>   # rename project overview files across registered projects
devme rm [path]                # remove a directory from the mesh index
devme rm [path] --delete       # remove from index and also delete the companion file
```

`devme init` registers the new file in the global index and updates navigation links in nearby companion files.

---

## Workflow

1. Run `devme init` in a directory you care about
2. Open `devme serve`
3. Browse to that directory in the interface
4. Update Status, Next Steps, and Session Log as work changes
5. Add annotations to files or folders that need extra context

---

## Interface

<!-- screenshot: close-up of the sidebar tree showing directory hierarchy with A-Z toggle and bookmarks panel -->

**Sidebar** — registered projects as a navigable directory tree, grouped by path. Drag the right edge to resize. Toggle between tree view and A–Z sort. Bookmarks panel at the bottom.

**Content area** — renders the companion file with full markdown support: tables with sortable columns, fenced code with syntax highlighting, collapsible sections. The Directory section renders as a live clickable file tree — every file and subfolder is a link you can open directly in the viewer or in your editor.

<!-- screenshot: annotation box below the page title — ✦ icon in header, note rendered as markdown below the filepath bar, ghost "✦ Add context" state visible on a folder in the directory tree -->

**Annotations** — the core feature. Attach a context note to any file or directory without modifying it. Click ✦ next to any item in the directory tree to add or edit a note. The note for the currently open page renders as a styled box below the title. Notes are stored in a separate JSON file and never written into your repos or folders.

<!-- screenshot: navigation section showing Up (purple), Nearby (amber), Down (green) color-coded links between project directories -->

**Navigation** — companion files link to each other by directory relationship. Parent directories appear in purple (↑), siblings in amber (↔), children in green (↓). The mesh self-assembles as you run `devme init` in new locations.

**Live reload** — an SSE connection watches all registered files and reloads the viewer automatically when any of them change.

---

## Sync

> **Optional.** This section only applies if you use the `whatdoing` project manager. Skip it if you don't.

devme integrates with [`whatdoing`](https://github.com/err404memory/whatdoing) project overview files. The canonical filename is `overview.md`; legacy names are still read for compatibility: `project.md`, `_OVERVIEW.md`, `PROJECT.md`, and `devme.md`. Those files usually hold the fuller project record. The companion file keeps the local context layer: navigation, annotations, session log, and a smaller view of current status.

When one of those overview files exists in a directory, `devme init` pulls its **Status** and **Next Steps** fields into the companion file automatically. After that you can keep them in sync:

```sh
devme update [path]   # pull latest from overview file into companion file
devme push [path]     # write Status/Next Steps changes back to overview file
devme watch [path]    # continuous two-way sync on file save
```

<!-- screenshot: Status card in the viewer showing project status pulled from whatdoing's project.md — Status and Next Steps rendered as a single bordered card -->

The project overview remains the source of truth. The companion file surfaces current context without duplicating the full project document.

If you do not use `whatdoing`, skip this section entirely. `devme` is complete without it.

---

## Logging

The `hooks/` directory contains an optional pipeline that scans local AI tool logs and terminal session logs and appends context to the Session Log in your companion files.

devme has no AI dependency. The hooks are log parsers. On terminal exit, `session-close` checks known locations for recently closed sessions from Claude Code, Kilo, Codex, Aider, Ghostty, tmux, or any other tool you configure. It reads those logs and extracts decisions, ideas, generated code, and session summaries, then appends that context to the companion file for the relevant project directory. No external service is contacted.

### How it works

```
terminal exit
  └─ session-close                   (bash EXIT trap — scans all configured tool dirs)
       ├─ parse-ai-session           (JSONL logs: Claude Code, Kilo, Codex, Aider, …)
       └─ parse-ghostty-session      (text logs: Ghostty, script, tmux, …)
            └─ update-session-docs   (appends context to companion files)
```

The summary is written into the `## Session Log` section of both the global hub file and the project companion file for whatever directory you were working in.

If you do not want shell hooks or session parsing, skip this section. The core product does not depend on it.

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

Set `vault_dir` in `~/.devme/config.json` to control where full session archives are saved (organised by `YYYY-MM/`). A path on a server mount keeps archives accessible across devices:

```json
"vault_dir": "~/server/sessions"
```

To add or override AI tool log directories, set `tool_paths` in `~/.devme/config.json`:

```json
"tool_paths": {
  "claude": "~/.claude/projects",
  "kilo":   "~/.kilo/projects",
  "codex":  "~/.codex/sessions"
}
```

Any tool directory that exists on your system will be scanned automatically. Remove a key to disable scanning that tool. See `hooks/session-close` for a list of supported tools and instructions for adding new ones.

The session detection window defaults to 300 seconds. Override with:

```sh
export DEVME_SESSION_THRESHOLD=600   # 10 minutes
```

### Dolphin integration

`hooks/devme-init` and `hooks/devme-open` are service menu helpers. They fork immediately so Dolphin doesn't freeze, use `notify-send` for desktop feedback, and resolve the `devme` binary from `$PATH`.

```sh
cp hooks/devme-init  ~/.local/bin/
cp hooks/devme-open  ~/.local/bin/
chmod +x ~/.local/bin/devme-init ~/.local/bin/devme-open
```

---

## Format

`devme init` generates a structured file with a fixed section schema:

````markdown
# project-name

## Navigation
- [Context Hub](~/.devme/me.md)
- Up: [parent-dir](/path/to/parent/me.md)
- Nearby: [sibling-project](/path/to/sibling/me.md)

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
````

**Navigation**, **Directory**, **AI Notes**, and **Change Log** are maintained automatically by the CLI. **Status**, **Next Steps**, and **Session Log** are yours — or synced from your project overview.

The AI Notes section automatically detects `CLAUDE.md`, session summary files, and other AI-generated context in the directory and links them here.

---

## Config

| Key | Default | Description |
|-----|---------|-------------|
| `username` | `"you"` | Displayed in the sidebar header |
| `filename` | `"me.md"` | Companion filename looked for in each directory |
| `hub_label` | `"Context Hub"` | Browser tab and sidebar title |
| `hub_dir` | `"~/.devme"` | Location of the global companion file and `serve.html` |
| `editor` | `"code"` | Editor launched by the "Open in …" button |
| `accent_color` | `"#7b96e8"` | Primary accent color throughout the UI |
| `timezone` | `"UTC"` | Timezone for timestamps |
| `notes_file` | `~/.devme/file-notes.json` | Where annotations are stored |
| `vault_dir` | `~/Documents/sessions` | Where full session archives are saved by the hooks |
| `server_prefix_local` | `""` | Local mount path for a remote filesystem (e.g. `~/server`) |
| `server_prefix_remote` | `""` | Corresponding path on the remote machine (e.g. `/srv/projects`) |
| `icon_folder` | built-in SVG | Closed folder icon — any SVG `url()` or data URI |
| `icon_folder_open` | built-in SVG | Open folder icon |
| `ann_icon_color` | accent color | Color of the ✦ annotation icon |

### Multi-device paths

If your filesystem is mounted across machines at different paths, set `server_prefix_local` and `server_prefix_remote`. Annotations are keyed to a device-agnostic path, so the same note is visible whether you are on the machine that owns the files or accessing them through a mount.

---

## Roadmap

- **Dolphin panel** — embed the viewer as a native preview pane in KDE Dolphin; context notes appear as you browse without opening a browser
- **Unified filesystem view** — single interface across local directories, mounted remotes, and SSH paths
- **File manager plugins** — layered integration with Nautilus, Thunar, and other managers as an alternative to the standalone server

---

## Support

This project is open-source and free to use.

If support or implementation help would be useful, open an issue titled `commercial support interest`.

---

## Audit

Run the repo audit before publishing or opening a PR:

```bash
./scripts/public-safety-audit.sh
```

If you want a local pre-commit guard for staged files:

```bash
./scripts/install-public-safety-hook.sh
```

The audit scans for generic home-path leaks, SSH/scp-style host references, obvious secret-key markers, and suspicious local artifact filenames. If a match is intentional, add a path glob to `.public-safety-allowlist`. If you want the audit to catch your own machine names or local path fragments, put custom regex patterns in `.public-safety-local-patterns` and keep that file local only.

---

## License

MIT
