# PhotoCraft skill for Claude

PhotoCraft (ArtCraft suite, storytold/photocraft) is a Photoshop-style image editor: PSD/PSB, layers, masks, smart objects, Camera Raw, actions. Every menu item, tool and dialog runs a command from **one registry of 500+ commands**; the UI, CLI, JSON control channel and MCP server call the same commands, so anything you can click an agent can do.

This repository packages a **skill** that teaches Claude to drive PhotoCraft (part of the [ArtCraft suite](https://github.com/storytold/photocraft) by storytold) and that **knows how to create the MCP server and guide you through it**, on **Windows and macOS**. It is unofficial and not affiliated with storytold.

> Part of the [ArtCraft Claude skills hub](https://github.com/Vinciux99/artcraft-claude-skills): all 13 skills, a picker and a one-command installer.

## What it does
- Operating manual for PhotoCraft: connection modes, ground rules, every MCP tool, recipes, CLI, control channel, command families and a safe workflow (work on copies, verify, save only with your OK).
- Built-in MCP setup assistant: Claude checks whether the server is connected, then walks you through installing it (automatic script or manual config), the launcher, authentication and verification.
- Troubleshooting table for the usual problems (server "disconnected", ports, macOS quarantine, Windows Store config location...).

## What you can ask Claude
- "Open `holiday.jpg`, apply a Curves adjustment layer for more contrast, and export a PNG copy."
- "Resize this photo to 1600 px wide, sharpen it, and save a JPEG next to the original."
- "Run the same grade on every image in `Pictures/raw` and put the results in `Pictures/graded`."
- "Create a 1080×1080 document with a gradient background and my text centred on it."
- "Export every layer of this PSD as a separate PNG."
- "Record an action that does X, then play it on these five files."
- "Show me the layer tree and rename the layers sensibly."
- "Apply Smart Sharpen at 80% and Gaussian Blur with radius 3 on a mask."

## Highlights
- Token-authenticated control channel and path-restricted automation (read/write roots) explained and set up for you.
- MCP tool table (headless and bridge), the CLI `run`/`batch`/`serve`, and the control-channel protocol.
- Background-job handling for filters, with `wait:false` and `jobs_list`.
- 500+ command families with documented examples (layers, adjustments, filters, text, actions, export).
- Built-in guidance for creating and troubleshooting the MCP server on Windows and macOS.

## App-specific notes
- Kit port **7878**; token file `~/.photocraft-control.token`; roots default to your **Pictures** folder (change them in the launcher and the headless MCP args if your images live elsewhere).
- Files must be under the roots; `file.*` path commands are refused over automation — use `doc_open`/`doc_save`/`doc_export`.
- Never share the token file.

## Get the skill

| Where | How |
|---|---|
| **Claude app** (claude.ai / Desktop) | Download [`photocraft-skill.zip](https://github.com/Vinciux99/photocraft-claude-skill/releases/latest/download/photocraft-skill.zip)`, then Customize > Skills > "+" > Create skill > Upload a skill. Needs Settings > Capabilities > "Code execution and file creation" on. |
| **Claude Code, plugin** | `/plugin marketplace add Vinciux99/artcraft-claude-skills` then `/plugin install photocraft@artcraft` |
| **Claude Code, manual** | copy `skills/photocraft/` to `~/.claude/skills/photocraft/` (the setup script below does it) |

## Create the MCP server

Prerequisite: install PhotoCraft from [upstream releases](https://github.com/storytold/photocraft/releases) (Windows `.msi`, macOS `.dmg`) and Claude Desktop and/or Claude Code.

**One command (recommended)**

Windows (PowerShell):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& ([scriptblock]::Create((irm https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.ps1))) -Apps photocraft"
```
macOS (Terminal):
```sh
curl -fsSL https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.sh | bash -s -- --apps photocraft
```
Add `-DryRun` (Windows) or `--dry-run` (macOS) to preview, and `-Mode live|headless|both` / `--mode ...` to choose the mode.

**Or from a clone:** double-click `setup-windows.cmd` (Windows) or `setup-macos.command` (macOS; first time right-click > Open).

**Or just ask Claude:** once the skill is installed, say "set up the PhotoCraft MCP server" and Claude will guide you step by step.

After setup: fully quit and reopen Claude, open PhotoCraft with the **"PhotoCraft (Claude)"** launcher, then ask Claude to use it. If the server shows "disconnected", the app was not running when Claude started: open it with the launcher and re-enable the server, or use headless mode.

What the setup changes (all backed up or reversible): Claude Desktop config (`mcpServers.photocraft`, backup `<file>.backup-<timestamp>`), the launcher, Claude Code's user-scope MCP list and `~/.claude/skills/photocraft/`.

## Repository layout
```
skills/photocraft/SKILL.md      the skill
app.json                   machine-readable settings (ports, paths, MCP args) used by the scripts
scripts/                   artcraft-setup.ps1 (Windows), artcraft-setup.sh (macOS), json-tool.js
setup-windows.cmd          double-click wrapper
setup-macos.command        double-click wrapper
.claude-plugin/plugin.json Claude Code plugin manifest
.github/workflows/ci.yml   validation, setup tests on Windows and macOS, release zip
```

## CI and releases
Every push validates `app.json`, `SKILL.md` and the scripts, then runs the setup scripts against a fake install on **Windows** and **macOS** runners. Pushing a tag `v*` builds `photocraft-skill.zip` and publishes a GitHub release.

## Status and caveats
- Windows paths and arguments come from a working install. The macOS paths (`/Applications/PhotoCraft.app`, CLI location and release asset name) follow the upstream packaging but have **not** been verified on a Mac; the skill marks unverified items and the setup script reports what it could not find.
- PhotoCraft is a young open-source project; behaviour can change between releases. `list_commands` / `photocraft-cli commands` is always the source of truth.
- Unofficial community packaging, MIT licensed (see `LICENSE`).
