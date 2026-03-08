# devme

A personal developer context mesh. One markdown file per project directory — browse, annotate, and navigate your entire codebase from a single local web interface.

## What it is

devme is a lightweight local dashboard for developers who keep notes alongside their code. Drop an `ash.md` file into any project directory and it becomes a node in your mesh — visible in the sidebar, browsable in the viewer, and linkable to everything else.

The viewer renders markdown with syntax highlighting, collapsible sections, sortable tables, and a live-reload connection that refreshes automatically when files change. A persistent annotation layer lets you attach context notes to any file or directory without modifying it.

## Structure

```
ash          CLI — register paths, serve, manage notes
serve.html   SPA template — the full browser interface
config.json  Personal config (not tracked)
```

## Setup

**Requirements:** Python 3.10+, a modern browser.

Copy `ash` to somewhere on your `$PATH`:

```sh
cp ash ~/.local/bin/ash
chmod +x ~/.local/bin/ash
```

Create `~/.ash/config.json`:

```json
{
  "username": "you",
  "filename": "ash.md",
  "hub_label": "My Mesh",
  "hub_dir": "~/.ash",
  "editor": "code",
  "accent_color": "#7b96e8",
  "timezone": "America/New_York"
}
```

Copy `serve.html` to `~/.ash/serve.html`.

## Usage

```sh
ash serve              # start the local server (default: localhost:7272)
ash register <path>    # add a directory to the mesh
ash new <path>         # create an ash.md in a directory
ash refresh            # rebuild the mesh index
ash rm [path]                # remove a directory from the mesh index
ash rm [path] --delete       # remove from index and delete the companion file
```

Open `http://localhost:7272` in your browser.

## Interface

**Sidebar** — your registered projects as a navigable tree. Drag the right edge to resize. Toggle A–Z sort or browse by directory structure. Bookmarks panel below the tree.

**Content area** — renders the current `ash.md` with full markdown support including tables, code blocks with syntax highlighting, collapsible sections, and directory trees. The filepath bar shows the current path with a copy button.

**Annotations** — every file and directory in the mesh can have a context note attached. Click ✦ next to a file link to add or edit. The note for the currently open page appears in the annotation box below the title and is editable via the ✦ icon in the title bar.

**Live reload** — an SSE connection watches for file changes and reloads automatically.

## Config options

| Key | Default | Description |
|-----|---------|-------------|
| `username` | `"ash"` | Displayed in the sidebar header |
| `filename` | `"ash.md"` | The context filename looked for in each directory |
| `hub_label` | `"Context Mesh"` | Browser tab and sidebar title |
| `hub_dir` | `"~/.ash"` | Directory of the global context file |
| `editor` | `"code"` | Editor opened by the "Open in …" button |
| `accent_color` | `"#7b96e8"` | Primary accent color throughout the UI |
| `timezone` | `"UTC"` | Timezone for timestamps |
| `icon_folder` | built-in SVG | Closed folder icon — any SVG `url()` or data URI |
| `icon_folder_open` | built-in SVG | Open folder icon |
| `ann_icon_color` | accent color | Color of the ✦ annotation icon |
| `vault_dir` | `~/Documents/sessions` | Where full session archives are saved by the hooks |

## Notes file

Annotations are stored in a JSON file separate from your markdown. Set the location in config:

```json
"notes_file": "~/.ash/file-notes.json"
```

This keeps notes out of your repos while making them portable across machines via sync.

## Session auto-logging

The `hooks/` directory contains scripts that close the loop between your terminal sessions and your companion files. On every terminal exit, your session is summarized by an AI and appended to the relevant companion file's Session Log automatically — no manual notes required.

### How it works

```
terminal exit
  └─ session-close          (bash EXIT trap)
       ├─ parse-ai-session  (Claude Code JSONL → summary via claude -p)
       └─ parse-ghostty-session  (terminal log → summary via claude -p)
            └─ update-session-docs  (appends summary to companion files)
```

The session summary is structured (Key Points, Decisions, Takeaway) and written into the `## Session Log` section of both the global hub and the project companion file for whatever directory you were working in.

### Install

```sh
# Copy all hook scripts to PATH
cp hooks/session-close        ~/.local/bin/
cp hooks/update-session-docs  ~/.local/bin/
cp hooks/parse-ai-session     ~/.local/bin/
cp hooks/parse-ghostty-session ~/.local/bin/
cp hooks/tmux-start-log       ~/.local/bin/
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

For tmux logging, add to `~/.tmux.conf`:

```
set-hook -g after-new-window   'run "~/.local/bin/tmux-start-log #{session_name} #{window_index} #{pane_id}"'
set-hook -g after-split-window 'run "~/.local/bin/tmux-start-log #{session_name} #{window_index} #{pane_id}"'
```

### Session log config

Add to `~/.ash/config.json`:

```json
"vault_dir": "~/server/sessions"
```

`vault_dir` is where full structured session archives are saved (organized by `YYYY-MM/`). Set it to a path on your server mount for cross-device access. The companion file Session Log gets a concise summary; the vault gets the full structured record.

The `session-close` threshold (how recently a Claude transcript must have been modified to count as "this session") defaults to 300 seconds. Override with:

```sh
export ASH_SESSION_THRESHOLD=600   # 10 minutes
```

### Dolphin integration

`hooks/ash-init` and `hooks/ash-open` are wrappers for use as Dolphin service menu actions. They fork immediately so Dolphin doesn't freeze, use `notify-send` for desktop feedback, and resolve the `ash` binary from `$PATH` rather than a hardcoded location.

```sh
cp hooks/ash-init  ~/.local/bin/
cp hooks/ash-open  ~/.local/bin/
chmod +x ~/.local/bin/ash-init ~/.local/bin/ash-open
```

## License

MIT
