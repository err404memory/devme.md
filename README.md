# devme

Context notes for your filesystem. Every directory gets a companion markdown file — attach notes to any file or folder, navigate your whole codebase from one local interface, and keep project status in sync with your project manager without touching anything it annotates.

---

> **Note:** Throughout this documentation, `me.md` is used as the companion filename placeholder.
> After running `devme install`, your actual configured filename (e.g. `alex.md`) will be used on your system.
> Wherever docs say `me.md`, substitute your configured filename.

## The idea

A well-organized filesystem is still opaque without context. What is this directory for? Why does this file exist? What was in progress when you last had this project open?

devme gives every directory a companion file (`me.md` by default, configurable). That file holds whatever context belongs to that location: status, next steps, a live directory tree, a change log, a session log. Each file links to its neighbors — parent, siblings, children — forming a navigable mesh across your whole filesystem.

The annotation layer is the core feature. You can attach a note to any file or directory in the interface without modifying it. The note is stored separately from your files, appears inline when you open that location in the viewer, and travels with the path across sessions. It's the equivalent of a sticky note on a manila folder in a filing cabinet — the folder's contents are unchanged, but the context is there every time you pull it out: what it's for, what's in progress, what to watch out for.

That same structure makes the files easy for AI assistants to read and maintain alongside you. The consistent section schema — Status, Next Steps, Change Log, Session Log — maps directly to how an AI assistant tracks project state, so your context mesh and your AI context stay naturally in sync.

<!-- screenshot: full interface — sidebar project tree on the left, rendered context file on the right showing status card and navigation links -->

---

## Structure

```
devme        CLI — init, serve, sync, manage
serve.html   Web interface (single-file SPA, loaded from ~/.devme/)
wizard.html  Setup wizard (browser-based, opened by devme install)
hooks/       Session auto-logging pipeline (optional, see below)
```

---

## Setup

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

`devme install` opens a browser-based setup wizard. It walks you through choosing your companion filename, editor, timezone, and accent color — with a live preview of the interface as you configure it.

On finish, the wizard creates `~/.devme/config.json`, installs bundled UI assets (`serve.html`, `wizard.html`) into `~/.devme/`, creates your global hub file, and writes a personalized `~/.devme/QUICKSTART.md`.

After setup, run `devme init` for at least one project directory, then launch `devme serve`. On first load, the interface starts a guided tour automatically. If you have no contexts yet, the tour shows the exact `devme init` command to run next. You can replay the walkthrough any time with the `Tour` button in the top bar.

---

## Usage

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

`devme init` registers the new file in your global index and back-propagates: neighboring companion files in parent and sibling directories automatically gain links to the new one.

---

## Platform Compatibility

- **Tested:** Linux
- **Likely works:** macOS (same POSIX path model as Linux)
- **Use WSL for now:** Windows native path handling in the web viewer is still being hardened

No specific shell is required for the core CLI (`devme ...`) — it runs as a normal Python command. 

Optional pieces have narrower support:
- `hooks/` scripts are Bash-oriented
- `hooks/devme-init` and `hooks/devme-open` are KDE Dolphin helpers (Linux-specific)

**Roadmap:** improve native Windows path handling and provide platform-specific helper script options.

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

## whatdoing sync

> **Optional.** This section only applies if you use the `whatdoing` project manager. Skip it if you don't.

devme integrates with [`whatdoing`](https://github.com/err404memory/whatdoing) project overview files. Canonical filename is `overview.md`; legacy names are still read for compatibility: `project.md`, `_OVERVIEW.md`, `PROJECT.md`, and `devme.md`. These are the detailed per-project documents maintained by whatdoing — tech stack, commands, status, roadmap, blockers — the full project record. The companion file is intentionally lighter: navigation, annotations, session log, and a live window into the overview's current status.

When one of those overview files exists in a directory, `devme init` pulls its **Status** and **Next Steps** fields into the companion file automatically. From that point you can keep them in sync:

```sh
devme update [path]   # pull latest from overview file into companion file
devme push [path]     # write Status/Next Steps changes back to overview file
devme watch [path]    # continuous two-way sync on file save
```

<!-- screenshot: Status card in the viewer showing project status pulled from whatdoing's project.md — Status and Next Steps rendered as a single bordered card -->

The project overview remains the source of truth. The companion file surfaces what matters now — without duplicating the documentation.

---

## Session auto-logging

The `hooks/` directory contains an optional pipeline that scans your local AI tool logs and terminal session logs — whatever exists on your system — and appends context to your companion files' Session Log automatically.

devme has no AI dependency. The hooks are log parsers. On terminal exit, `session-close` checks known locations for recently-closed sessions from Claude Code, Kilo, Codex, Aider, Ghostty, tmux, or any other tool you configure. It reads those logs and extracts what was discussed — decisions, ideas, generated code, session summaries — then appends that context to the companion file for the relevant project directory. No external service is contacted. Nothing is sent anywhere.

### How it works

```
terminal exit
  └─ session-close                   (bash EXIT trap — scans all configured tool dirs)
       ├─ parse-ai-session           (JSONL logs: Claude Code, Kilo, Codex, Aider, …)
       └─ parse-ghostty-session      (text logs: Ghostty, script, tmux, …)
            └─ update-session-docs   (appends context to companion files)
```

The summary is written into the `## Session Log` section of both the global hub file and the project companion file for whatever directory you were working in.

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

## Companion file format

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

## Config reference

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

If your filesystem is mounted across machines at different paths (e.g. a server mounted via rclone), set `server_prefix_local` and `server_prefix_remote`. Annotations are keyed to a device-agnostic path and resolve correctly on each machine — the same note is visible whether you're on the machine that owns the files or accessing them through a mount.

---

## Roadmap

- **Dolphin panel** — embed the viewer as a native preview pane in KDE Dolphin; context notes appear as you browse without opening a browser
- **Unified filesystem view** — single interface across local directories, mounted remotes, and SSH paths
- **File manager plugins** — layered integration with Nautilus, Thunar, and other managers as an alternative to the standalone server

---

## Changelog

### 2026-03-08

**Preview migration note** — users coming from early pre-release `ash` builds can run `devme install --force` to refresh assets and write config into `~/.devme/`. Existing legacy config is still read automatically during transition.

**Session auto-logging pipeline** — added `hooks/` with 7 scripts: `session-close` (bash EXIT trap), `parse-ai-session` (Claude Code JSONL → structured summary), `parse-ghostty-session` (terminal log → structured summary), `update-session-docs` (appends summaries to companion files), `tmux-start-log` (tmux pipe-pane logging), `devme-init` and `devme-open` (Dolphin service menu wrappers). Every terminal session is now automatically summarized and written into the Session Log of the relevant companion file.

**`devme rm`** — new subcommand removes a directory from the global mesh index. `--delete` flag also removes the companion file from disk.

**Visible status placeholders** — freshly initialized companion files now show `_No status set._` and `_No next steps defined._` instead of invisible HTML comments that rendered as blank cards.

**Sort button** — toggles between `A–Z` and `Tree` labels, making the action clear in both states.

**`_upgrade_file`** — corrected return type annotation from `-> None` to `-> bool`.

**README** — full rewrite: corrected CLI command names, added sticky note concept framing, configurable filename explanation, companion file format example, whatdoing sync section, multi-device path config, roadmap, and screenshot placeholders.

**MANUAL.md** — added full onboarding manual covering installation, configuration, first run, daily workflow, annotations, companion file format, whatdoing integration, multi-device setup, and troubleshooting.

**`config.example.json`** — added example config file with all supported keys and inline instructions.

---

## Support

This project is open-source and free to use.

Optional paid support and implementation services may be introduced in the future for teams that want faster setup, migration, and workflow customization.

If that would be useful, open an issue titled `commercial support interest`.

Direct support links will be added here once funding channels are set up.

---

## Public Safety Check

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
