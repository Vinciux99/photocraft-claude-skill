---
name: photocraft
description: Edit raster images and layered documents with PhotoCraft (Photoshop-style, ArtCraft suite) through MCP, the CLI or the control channel, and set up, connect or troubleshoot its MCP server on Windows and macOS.
---

# PhotoCraft for Claude

PhotoCraft (ArtCraft suite, storytold/photocraft) is a Photoshop-style image editor: PSD/PSB, layers, masks, smart objects, Camera Raw, actions. Every menu item, tool and dialog runs a command from **one registry of 500+ commands**; the UI, CLI, JSON control channel and MCP server call the same commands, so anything you can click an agent can do.

Compared with its sister apps, PhotoCraft has two extra safety layers: an **authenticated** control channel (token) and **closed file access** (read/write roots).

## Quick check: is the MCP server connected?

1. Look at the tools you have. If tools from the MCP server `photocraft` are available (for example `list_commands`, `execute` or `inspect_*` tools) the server is connected: skip to "Operating knowledge" below and use them directly.
2. If they are missing, check whether the user installed it under the name `photocraft` or `photocraft-headless`. In Claude Desktop the server may show as **disconnected**.
3. **Most common cause of "disconnected":** the live server (`mcp --bridge 127.0.0.1:{port} --control-token-file {home}/.photocraft-control.token`) exits when PhotoCraft is not running at the moment Claude starts. Fix: open PhotoCraft with the **"PhotoCraft (Claude)" launcher** (it starts the app with its control channel on), then re-enable the server in Claude's settings or fully quit and reopen Claude.
4. If the user does not need to watch the app live, **headless mode** always starts (no window, works on files). See "Live, headless or both".
5. If nothing is configured yet, follow "Set up the MCP server" below. Ask the user which OS they use if you do not know.

## Set up the MCP server (guide the user)

Walk the user through this step by step. Ask for the OS (Windows or macOS) first. If you have a shell on the user's machine you may run the steps yourself, but **ask before editing any config file and always back it up first**. Never print tokens or keys in chat.

### 1. Prerequisites
- Install PhotoCraft from its upstream releases page: https://github.com/storytold/photocraft/releases (Windows: the `.msi` installer; macOS: the `.dmg`, then drag the app to `/Applications`).
- Install Claude Desktop and/or Claude Code. For Claude Desktop, fully quit and reopen it after any config change.
- The MCP server is the **CLI** that ships with the app: `photocraft-cli`. Windows installs it next to the app (`C:\Program Files\PhotoCraft\photocraft-cli.exe`, not on `PATH`). On macOS the CLI is a separate release download (asset `photocraft-cli-<version>-macos-universal.zip`) unless it is bundled in the app (unverified): the setup script below fetches it into `~/.artcraft/bin/` and clears the macOS quarantine flag.
- This kit uses port **7878** (the upstream default), which does not clash with the other apps of the kit.

### 2. Option A: automatic setup (recommended)
The setup script finds the app and CLI, creates the launcher, backs up and updates Claude Desktop's config, registers the server in Claude Code (if the `claude` command exists) and installs this skill. Preview first with a dry run: nothing is changed.

Windows (PowerShell):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& ([scriptblock]::Create((irm https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.ps1))) -Apps photocraft"
```
macOS (Terminal):
```sh
curl -fsSL https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.sh | bash -s -- --apps photocraft
```
Or clone/download this repository and double-click `setup-windows.cmd` (Windows) or `setup-macos.command` (macOS; first time: right-click, Open). Useful options: `-Mode live|headless|both` / `--mode ...` (default `live`; `both` registers `photocraft` live and `photocraft-headless`), `-DryRun` / `--dry-run`, `-NoLauncher`, `-NoClaudeDesktop`, `-NoClaudeCode`, `-NoSkill` (macOS: `--no-launcher`, `--no-claude-desktop`, `--no-claude-code`, `--no-skill`).

What it changes: Claude Desktop config (`%APPDATA%\Claude\claude_desktop_config.json` plus the Microsoft Store/MSIX copy under `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\`; macOS `~/Library/Application Support/Claude/claude_desktop_config.json`; backup saved next to it as `<file>.backup-<timestamp>`), the launcher "PhotoCraft (Claude)" (Windows: Desktop shortcut; macOS: `~/Applications/PhotoCraft (Claude).app` plus a Desktop alias), Claude Code's user-scope MCP list, and `~/.claude/skills/photocraft/`.

### 3. Option B: manual setup
**Windows** - edit `%APPDATA%\Claude\claude_desktop_config.json` (if Claude was installed from the Microsoft Store, also the copy under `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\`). Keep other servers; add this inside `mcpServers`. Backslashes must be doubled in JSON.
```json
{
  "mcpServers": {
    "photocraft": {
      "command": "C:\\Program Files\\PhotoCraft\\photocraft-cli.exe",
      "args": [
        "mcp",
        "--bridge",
        "127.0.0.1:7878",
        "--control-token-file",
        "C:\\Users\\YOUR_NAME\\.photocraft-control.token"
      ]
    },
    "photocraft-headless": {
      "command": "C:\\Program Files\\PhotoCraft\\photocraft-cli.exe",
      "args": [
        "mcp",
        "--automation-read-root",
        "C:\\Users\\YOUR_NAME\\Pictures",
        "--automation-write-root",
        "C:\\Users\\YOUR_NAME\\Pictures"
      ]
    }
  }
}
```
**macOS** - edit `~/Library/Application Support/Claude/claude_desktop_config.json`. Use the absolute CLI path (replace `YOUR_NAME`; JSON does not expand `~`). The CLI path is unverified if your install differs: check with `ls ~/.artcraft/bin /Applications/PhotoCraft.app/Contents/MacOS`.
```json
{
  "mcpServers": {
    "photocraft": {
      "command": "/Users/YOUR_NAME/.artcraft/bin/photocraft-cli",
      "args": [
        "mcp",
        "--bridge",
        "127.0.0.1:7878",
        "--control-token-file",
        "/Users/YOUR_NAME/.photocraft-control.token"
      ]
    },
    "photocraft-headless": {
      "command": "/Users/YOUR_NAME/.artcraft/bin/photocraft-cli",
      "args": [
        "mcp",
        "--automation-read-root",
        "/Users/YOUR_NAME/Pictures",
        "--automation-write-root",
        "/Users/YOUR_NAME/Pictures"
      ]
    }
  }
}
```
**Launcher** (starts the app with the control channel; needed before the live server can connect):
- Windows: create a Desktop shortcut named "PhotoCraft (Claude)" with target `"C:\Program Files\PhotoCraft\photocraft.exe" --control 7878 --control-token-file "C:\Users\YOUR_NAME\.photocraft-control.token" --automation-read-root "C:\Users\YOUR_NAME\Pictures" --automation-write-root "C:\Users\YOUR_NAME\Pictures"`
- macOS: run `open -n -a "/Applications/PhotoCraft.app" --args --control 7878 --control-token-file "/Users/YOUR_NAME/.photocraft-control.token" --automation-read-root "/Users/YOUR_NAME/Pictures" --automation-write-root "/Users/YOUR_NAME/Pictures"`, or save it as an app with Script Editor/`osacompile` (the setup script does this for you).

**Claude Code** (any OS; run once):
Windows (PowerShell):
```powershell
claude mcp add --scope user photocraft -- "C:\Program Files\PhotoCraft\photocraft-cli.exe" mcp --bridge 127.0.0.1:7878 --control-token-file C:\Users\YOUR_NAME\.photocraft-control.token
claude mcp add --scope user photocraft-headless -- "C:\Program Files\PhotoCraft\photocraft-cli.exe" mcp --automation-read-root C:\Users\YOUR_NAME\Pictures --automation-write-root C:\Users\YOUR_NAME\Pictures
```
macOS (Terminal):
```sh
claude mcp add --scope user photocraft -- "$HOME/.artcraft/bin/photocraft-cli" mcp --bridge 127.0.0.1:7878 --control-token-file /Users/YOUR_NAME/.photocraft-control.token
claude mcp add --scope user photocraft-headless -- "$HOME/.artcraft/bin/photocraft-cli" mcp --automation-read-root /Users/YOUR_NAME/Pictures --automation-write-root /Users/YOUR_NAME/Pictures
```
Check with `claude mcp list`.

### 4. Authentication
The app creates a 256-bit token file at `~/.photocraft-control.token` (Windows: `%USERPROFILE%\.photocraft-control.token`) when it starts with `--control-token-file`. The MCP bridge reads that same file. Never print the token in chat. File access is confined to the Pictures folder by the automation roots in the launcher.

### 5. Finish and verify
1. **Fully quit and reopen Claude** (Desktop: quit from the tray/menu bar, not just close the window).
2. Open PhotoCraft with the **"PhotoCraft (Claude)" launcher** before asking Claude to use it.
3. In Claude, confirm the server `photocraft` is connected and call its `list_commands` tool (or equivalent). A non-empty command list means it works.

### Live, headless or both
- **Live** (`photocraft`): drives the open app; the user sees every change and screenshots/UI tools work. Needs the launcher first.
- **Headless** (`photocraft-headless`): in-process engine, no window; always starts; works on files; UI/screenshot tools are not available.
- **Both**: register both and pick per task. Switch with `-Mode` / `--mode`, or by editing the config entries above.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Server shows "disconnected" right after Claude starts | Live server exits when PhotoCraft is not running | Open PhotoCraft with the "PhotoCraft (Claude)" launcher, then re-enable the server or restart Claude; or use headless mode |
| "Port already in use" / app opens but Claude cannot connect | Another app or an old PhotoCraft instance holds port 7878 | Quit all PhotoCraft windows, check the launcher uses the kit port, make sure no other ArtCraft app uses the same port |
| `command not found` / spawn error | CLI path wrong or the CLI is not installed (Windows: not on `PATH`; macOS: separate download) | Use the full absolute path in the config; macOS: run the setup script (it downloads the CLI to `~/.artcraft/bin/`) |
| macOS: "cannot be opened because the developer cannot be verified" | Gatekeeper/quarantine on the app or CLI | Right-click the app, Open; for the CLI: `xattr -dr com.apple.quarantine ~/.artcraft/bin/photocraft-cli` |
| `unauthorized` / token or key errors | Auth file missing or stale (token-file) | Restart the app with the launcher so it writes a fresh file; never paste the secret in chat |
| Config edited but nothing changed (Windows) | Claude installed from the Microsoft Store reads the MSIX copy of the config | Edit/update `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\claude_desktop_config.json` too |
| JSON config error / server not listed | Unescaped backslashes or trailing comma | Double the backslashes on Windows; validate the JSON; restore the `.backup-*` file if needed |
| App crashes on start or window is blank | Graphics backend problem | Windows: set `WGPU_BACKEND=vulkan` (or `dx12`) for one launch; avoid `gl` on some AMD drivers |
| Changes do not appear in the window | You are on the headless server | Use the live server (launcher first) |
| Quotes disappear in PowerShell JSON arguments | Windows PowerShell 5.1 quoting for `.exe` | Use `--script`/files where the CLI supports them, or single-quote the whole JSON |

## Operating knowledge

## Interfaces

| Interface | When | Window? |
|---|---|---|
| **MCP** `photocraft-cli mcp` | Claude and MCP clients | No (headless) or yes with `--bridge` |
| **CLI** `photocraft-cli run / batch / ...` | One-off operations and folders of images | No |
| **serve** `photocraft-cli serve` | Many fast edits over JSON lines on stdio or TCP, without MCP framing | No |
| **Control channel** `photocraft --control 7878` | See and drive the live app | Yes |

### What is different from the sister apps
- **The control channel is authenticated** with a 256-bit token. The first request on every TCP connection must be `{"id":"auth","method":"auth","params":{"token":"<64 hex>"}}`.
- **File access is closed by default.** You need `--automation-read-root` / `--automation-write-root`. Paths must be inside the root (relative with `/` or absolute under the root); `..`, other drive prefixes and links that escape are rejected. The destination folder of a new file must already exist.
- **`file.*` commands that take paths are refused over automation** (`file.open`, `file.save`, `file.saveAs`, `file.saveACopy`, `layer.exportAs {path}`, preset imports...). Open/save with `app.open` / `app.save` (channel) or `doc_open` / `doc_save` / `doc_export` (MCP). `file.new`, `file.close*` and a few path-less ones remain allowed.
- **Long commands are background jobs:** all `filter.*`, `edit.contentAwareFill`, `edit.contentAwareScale`, `file.automate.photomerge`, `brush.presets.importAbr`. By default the call waits; with `"wait": false` you get `{job, pending: true}`; poll `jobs.list`, cancel with `jobs.cancel`.

### Recommended live setup (bridge)
1. Start the app with a token file and roots — this is what the "PhotoCraft (Claude)" launcher does:
   - Windows: `"C:\Program Files\PhotoCraft\photocraft.exe" --control 7878 --control-token-file "%USERPROFILE%\.photocraft-control.token" --automation-read-root "%USERPROFILE%\Pictures" --automation-write-root "%USERPROFILE%\Pictures"`
   - macOS: `open -n -a /Applications/PhotoCraft.app --args --control 7878 --control-token-file "$HOME/.photocraft-control.token" --automation-read-root "$HOME/Pictures" --automation-write-root "$HOME/Pictures"` (unverified)
   If the token file does not exist PhotoCraft creates it (on Windows protect it with a user-only ACL; on macOS `chmod 600`). With neither token nor file it generates a session token and prints it to stderr. Environment equivalents: `PHOTOCRAFT_CONTROL_TOKEN`, `PHOTOCRAFT_CONTROL_TOKEN_FILE`, `PHOTOCRAFT_AUTOMATION_READ_ROOT`, `PHOTOCRAFT_AUTOMATION_WRITE_ROOT` (the root variables apply only to the desktop app).
2. MCP server: live `mcp --bridge 127.0.0.1:7878 --control-token-file <token file>`; headless `mcp --automation-read-root <dir> --automation-write-root <dir>`. In bridge mode the roots are configured on the *app* process, not on the CLI. In headless (`mcp`, `serve`) they must be flags (environment variables do not apply).
3. **Security:** the token gives full control of the app (UI input, commands). Never paste it in chat, logs or versioned files; do not pass `--control-token` in clear text on shared machines; the channel is loopback-only and unencrypted.

### Choosing the channel
1. A PhotoCraft MCP server is available → use it (bridge to see edits live, headless for file jobs).
2. A shell on the user's machine → the CLI (`run`, `batch`, `serve`).
3. A cloud-sandbox shell is a different machine: it can read/write its own files but cannot run the user's app or reach its localhost port. It can prepare JSON files (actions, parameters).
4. Last resort: computer use on the UI (slow). Mind shortcuts with `[` `]` on non-US keyboards.

## MCP tools

| Tool | What it does | Mode |
|---|---|---|
| `command_list {filter?, enabled_only?}` | Bare array of commands `{id, label, menu, shortcut, params (text doc), enabled, disabledReason, needsDocument}` | both |
| `command_run {id, params?, wait?}` | Run a command (`params` must be a JSON object) | both |
| `command_batch {steps:[{id, params, wait?}], stop_on_error}` | Several commands in order (max 256 steps) | both |
| `jobs_list` / `jobs_cancel {job?}` | Background jobs | both |
| `doc_new {...}` | = `file.new` (`width`, `height`, `mode`, `depth`, `background`) | both |
| `doc_inspect` | = `document.inspect`: layer tree, history, snapshots, selection, channels | both |
| `doc_open {path}` | Open a file under the read root | both |
| `doc_place {path, scale?, fit?, center?}` | Place a file as a smart object above the active layer | both |
| `doc_save {path?, tgaBits?}` / `doc_export {path, tgaBits?}` | Save (native `.pcraft`, PSD/PSB; otherwise export by extension: png, jpg, tiff, tga...) | both |
| `doc_export_layers {dir, format?, prefix?, visibleOnly?}` | One file per layer | both |
| `doc_export_data_sets` / `doc_import_data_sets` | Variables and data sets (CSV) | both |
| `doc_render_preview {max_side?, index?}` | Preview PNG (max 2048 px; in bridge it is a window screenshot) | both |
| `doc_select {index}` / `doc_close {index?}` | Active document / close | headless only |
| `session_list`, `ui_inspect` | Open documents; full UI state | bridge |
| `ui_screenshot {max_side?}` | Window screenshot | bridge |
| `ui_pointer {events, modifiers?, button?}` | Use the active tool with `down/move/up` events in document coordinates (pressure, tilt: simulates a pen) | bridge |
| `ui_menu_invoke {id}` | Like a menu click (may open the dialog) | bridge |
| `ui_set {fields}` | Tool, panels, zoom, theme, brush size... | bridge |
| `control_call {method, params}` | Any control-channel method | bridge |

Pure reads (`ui_inspect`, `doc_inspect`, `command_list`, `jobs_list`, `session_list`, `ui_screenshot`) are retried automatically; **edits are not**: after a timeout or transport error, check the document (`doc_inspect`) before repeating. Client timeout is 55 s: for long commands use `wait: false` + `jobs_list`.

## CLI
```sh
# Open, edit, save in one call
photocraft-cli run wave.psd \
  --cmd filter.sharpen.smartSharpen     --params '{"amount":80}' \
  --cmd layer.newAdjustmentLayer.curves --params '{"points":[[0,0],[64,48],[192,212],[255,255]]}' \
  --out wave-final.png

# A list of actions over a folder of images
photocraft-cli batch --actions grade.json --in ./raw --out ./graded

# Every subcommand explains itself
photocraft-cli batch --help

# MCP (headless or bridge) and the JSON-lines server
photocraft-cli mcp
photocraft-cli serve --automation-read-root /work --automation-write-root /work
```
Run `photocraft-cli --help` and `<subcommand> --help` before use. The `--actions` format is the `steps` of `actions.get`: `[[id, params], ...]`.

### serve (JSON lines)
Methods: `engine.execute {command, params?, wait?}`, `jobs.list`, `jobs.cancel {job?}`, `engine.commands {filter?}`, `session.list`, `doc.open {path}`, `doc.new {...}`, `doc.place`, `doc.exportLayers`, `doc.exportDataSets`, `doc.importDataSets`, `doc.save {path?, format?, quality?, index?, tiffLayers?, tgaBits?}`, `doc.inspect {index?}`, `doc.render {index?, maxSide?, path?}`, `doc.select {index}`, `doc.close {index?}`, `batch {steps:[{command, params?, wait?} | {method, params?}], stopOnError?}`, `methods`.
```sh
printf '%s\n' \
  '{"id":1,"method":"doc.open","params":{"path":"in.jpg"}}' \
  '{"id":2,"method":"batch","params":{"steps":[{"command":"image.adjustments.invert"},{"command":"filter.blur.gaussianBlur","params":{"radius":3}}]}}' \
  '{"id":3,"method":"doc.save","params":{"path":"out.png"}}' | \
  photocraft-cli serve --automation-read-root /work/project --automation-write-root /work/project
```
On stdio `auth` is not needed; with `--port` it is. A layered save (PSD/PSB/.pcraft) marks the document saved; a flat export is a copy (the document stays `dirty`).

## Control channel (`photocraft --control 7878`)
TCP `127.0.0.1`, one JSON line per request/response. First line: `auth`. Then `{"id":1,"method":"...","params":{}}` → `{"id":1,"ok":true,"result":...}` or `{"ok":false,"error":"..."}`. Limits: request 1 MiB, response 8 MiB, 16 connections, idle connections closed after 30 s, reply within 60 s. `timeout: not run` = safe to resend; `timeout: no reply in 60s` = may have been applied, check first.

| Method | Use |
|---|---|
| `engine.execute {command, params, wait?}` | Any command, never a dialog (also UI commands like `view.zoomIn`, `window.theme.pro`) |
| `engine.commands` | All commands (bare array, no filter on desktop) |
| `ui.menu.invoke {id, params?, wait?}` / `ui.menu.list` | Menu click (opens the dialog if no params); menu tree. `window.document.<i>` brings a document forward |
| `ui.inspect` | Full UI state: tool, panels, views, dialogs, windows, `toolOptions`, `jobs`, `notices`, `perf` |
| `ui.set {tool?, zoom?, center?, rotation?, fit?, theme?, brushSize?, selectionMode?, panels?, dockTabs?, dock?, dockWidth?, maskTarget?, shapeStroke?, gradientBlendMode?, cropOverlay?, eyedropperSampleSize?, ...}` | Change UI state; unknown fields = error, nothing applied. `zoom` 1 = 100%; `theme` `pro\|proMedium\|studio\|studioLight\|classic` |
| `ui.dialog.open {kind, fields?}` / `ui.dialog.set {dialog, field, value}` / `ui.dialog.confirm {dialog, wait?}` / `ui.dialog.cancel {dialog}` | Dialogs: `newDocument`, `layerStyle {effect?}`, `colorPicker {target}`, `command {command}`, `newCropPreset`... The unsaved-changes prompt is `"discard"` with `choice: save\|dontSave\|cancel` |
| `ui.pointer {events:[{kind: down\|move\|up, x, y, pressure?, tiltX?, tiltY?, rotation?}], tool?, modifiers?, button?}` | Drive the tool in document coordinates. `command: true` = temporary Move; right button opens context menus (`ui.context.choose {id}`) |
| `ui.click {x, y, button?, count?}`, `ui.move`, `ui.key {key, command?, shift?, alt?, ctrl?}`, `ui.type {text}` | Input in screen points |
| `ui.window.open {document?}` / `ui.window.close`, `ui.resize`, `ui.focus` | Windows |
| `ui.screenshot {path?, focus?}` | PNG of the main window (base64 without path; path relative to the write root) |
| `app.open {path}` / `app.save {path?, tgaBits?}` | File I/O through the roots; reply with `warnings` |
| `jobs.list` / `jobs.cancel {job}` | Jobs |
| `app.quit` | Quit |

## Main commands (documented examples)
Use `command_list {filter}` to find more (500+) and read the `params` doc first. Almost all need an open document (`needsDocument: true`): create or open one first.

**Document:** `file.new {width, height, mode:"rgb", depth:8, background:"white"|"transparent"}`, `document.inspect`, `document.pixel {x, y}`, `document.sampleColor {x, y, size, sampleLayer}`, `document.presets.list|save|get|delete`, `document.recent.list|add`, `image.crop`, `image.adjustments.invert`, `history.selectSnapshot`.
**Layers:** `layer.new.layer {name}`, `layer.select {layer, mode: replace|toggle|range|add}`, `layer.setProps {layer?, name, visible, opacity 0..1, blend:"Multiply"}`, `layer.setExpanded`, `layer.hideLayers|showLayers {layer}`, `layer.newAdjustmentLayer.curves {points}`, `layer.newAdjustmentLayer.hueSaturation {hue, saturation}`, `layer.setAdjustment`, `layer.newFillLayer.pattern`, `layer.fill.solidColor.set {color}`, `layer.fill.pattern.set`, `layer.layerMask.edit {density, feather}`, `layer.layerStyle.replace`, `layer.layerStyle.makeDefault|defaultFor`, `style.presets.new {effects}`, `layer.rasterize.*`, `layer.smartObjects.editContents|convertToLayers|saveContents`, `layer.smartFilter.setParams`.
**Channels / selections:** `channel.target`, `channel.setVisible`, `channel.list`, `select.rect {x, y, width, height, mode, ellipse}`, `select.magneticLasso {points}`.
**Painting / tools:** `paint.stroke {points:[[x,y,pressure],...], size, color, mode?}`, `paint.pencil`, `paint.gradient`, `paint.contentAwareMove`, `tools.setBrush {preset?}`, `tools.setColors`, `tool.presets.new|list|edit {tool}`, `gradient.presets.select`, `shape.create`, `shape.edit`, `edit.fill {contents, color, pattern, mode, opacity, preserveTransparency}`, `edit.transform.warp {style, bend}`, `edit.contentAwareFill`, `edit.contentAwareScale`.
**Filters (background jobs):** `filter.sharpen.smartSharpen {amount}`, `filter.blur.gaussianBlur {radius}`, `filter.distort.displace`, `filter.cameraRaw` (the full dialog is drivable via `ui.menu.invoke {id:"filter.cameraRaw", params:{ui:{set, view, scope, commit|cancel}}}`).
**Type:** `type.editText`, `type.edit`, `type.hitTest {layer?, x, y}`, `type.caret {layer?, index}`, `type.navigate {index, move}`.
**Actions:** `actions.list`, `actions.get {action}` (→ `steps: [[id, params], ...]`), `actions.record {name}`, `actions.stop`, `actions.play {action, from?}` (succeeds even with a failed step: check `failed`), `actions.move`, `actions.delete`, `actions.toggle`, `actions.rename`, `actions.insert`, `actions.dialog`, `actions.conditional {if, then, else}`, `actions.assignSet`, `file.automate.batch`.
**Export:** `file.export.saveForWebLegacy` (dialog), `file.export.layersToFiles`, `file.export.dataSetsAsFiles`, `file.import.variableDataSets`, `file.placeEmbedded` (use the `doc_*` versions instead).
**Preferences / UI:** `prefs.get {path}`, `prefs.set {path, value}` or `{values:{...}}`, `prefs.reset`, `edit.keyboardShortcuts {set, reset, removeConflicts, importKys}`, `edit.menus`, `edit.toolbar`, `edit.colorSettings`, `color.profileMismatch {action}`, `view.fitOnScreen`, `view.actualPixels`, `view.zoomIn|zoomOut`, `window.panel.move|float|dock|activate|close|layout|group`, `help.systemInfo`.

Settings live in `preferences.json` (Windows `%APPDATA%\Photocraft\`; macOS `~/Library/Application Support/Photocraft/` unverified; override with `PHOTOCRAFT_CONFIG_DIR`); autosaves in a `Recovery` folder next to it; brush presets and `actions.json` under `Presets`.

## Recommended workflow
1. Orient: `session_list` / `ui_inspect` (bridge) or `doc_inspect`.
2. Open or create: `doc_open {path}` (under the read root) or `doc_new`.
3. Find commands: `command_list {filter:"blur"}`; read `params`, check `enabled`/`disabledReason`.
4. Edit with `command_run` or `command_batch`; long filters with `wait:false` + `jobs_list`.
5. Verify with `doc_render_preview` (or `ui_screenshot`) and `doc_inspect`.
6. Save with `doc_save` (PSD/.pcraft) or `doc_export {path:"out.png"}` under the write root, when the user agrees. Never overwrite originals without asking: export a copy next to them.
