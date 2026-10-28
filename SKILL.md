---
name: figcraft
description: "Use this skill when you need to drive FigCraft (open-source Figma alternative) via its MCP server. Covers creating and editing design layers, running any menu command, inspecting the document as JSON, exporting to PNG/SVG/PDF, working with components and variants, auto layout, variables, prototypes, and live collaboration sessions. Requires figcraft-cli to be installed and running. Do NOT use for reading/writing other file formats or for contributing to the FigCraft codebase itself."
---

# FigCraft MCP Skill

## Overview

FigCraft exposes a full MCP server through `figcraft-cli mcp`. Every action
available in the UI — frames, auto layout, components, variants, styles,
variables, prototypes, exports — is a command an agent can call over MCP.

## Prerequisites

Build the CLI from the repo, or download a release binary:

```
git clone https://github.com/storytold/figcraft
cd figcraft
cargo build --release -p figcraft-cli
# binary: target/release/figcraft-cli.exe  (Windows)
```

## Starting the MCP Server

```powershell
# Headless (no window, in-process document)
figcraft-cli mcp

# Attached (drives the running FigCraft app live)
# First start the app: figcraft --control 7979
figcraft-cli mcp --attach 7979

# Confine file access to specific folders (recommended)
figcraft-cli mcp --automation-read-root C:\designs --automation-write-root C:\designs\out
```

## Binary location

- CLI: `C:\Program Files\FigCraft\figcraft-cli.exe`
- App: `C:\Program Files\FigCraft\figcraft.exe`

If the `figcraft` MCP tools are not available, tell the user to register the
server (`figcraft-cli.exe mcp`) in their agent's MCP config. Do not search for
`.mcp.json`; each agent has its own config file.

## Core Conventions

- **Coordinates**: page units (1 unit = 1 px at 100% zoom), x right, y down.
- `x`/`y` on `create_node` and `set_properties` are **relative to the parent frame** (or the page when there is none).
- **Colours**: `"#RRGGBB"` or `"#RRGGBBAA"`. Gradients: `{"type":"linearGradient","stops":["#hex","#hex"]}`.
- **Every menu action is a command** — use `list_commands` to discover them.
- New layers automatically become the selection.
- Every mutation is one undo step; `batch` makes several into one.
- Verify your work: `inspect_document` / `get_node` for numbers, `screenshot` for pixels.
- Programmatic calls never open dialogs.

## Essential Tools

| Tool | Purpose |
|---|---|
| `list_commands {filter?}` | Discover all 400+ commands with params |
| `run_command {command, params}` | Run any command (align, layout, component, export…) |
| `batch {commands, label?}` | Several commands as one undo step |
| `create_node {type, x, y, width, height, parent, name, text, props}` | Create frame, rect, ellipse, text, vector, component, section… |
| `set_properties {ids, props}` | Set geometry, fills, strokes, auto layout, text… |
| `inspect_document {depth, allPages}` | Full document as JSON |
| `get_node {id, depth}` | One layer as JSON |
| `select {ids}` | Set the selection |
| `export {ids, format, scale, path}` | Export as png, jpg, svg, webp, pdf, fig |
| `screenshot {ids?, scale?, maxSize?}` | PNG of the canvas or specific layers |
| `open_file {path}` / `save_file {path?}` | Open .figcraft, .fig, .svg, .json; save |
| `undo` / `redo` | Step through history |
| `pointer {kind, x, y, ...}` | Mouse gestures (click, drag, double…) |
| `key {key, shift, alt, cmd}` | Keyboard shortcuts |
| `type_text {text, id?}` | Type into a text layer |

## Common Workflows

### Create a frame with a rectangle inside

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"create_node","arguments":{"type":"frame","x":0,"y":0,"width":375,"height":812,"name":"Screen"}}}
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"create_node","arguments":{"type":"rectangle","x":16,"y":24,"width":343,"height":48,"parent":"<id from above>"}}}
{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"set_properties","arguments":{"ids":["<rect id>"],"props":{"fills":[{"type":"solid","color":"#1098AD"}],"cornerRadius":8}}}}
```

### Start from a template

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"run_command","arguments":{"command":"file.newFromTemplate","params":{"template":"mobile-app"}}}}
```

Available templates: `mobile-app`, `landing-page`, `dashboard`, `design-system`, `wireframe-kit`, `social-posts`, `slides`, `icon-set`.

### Export a frame to PNG

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"export","arguments":{"ids":["<frame id>"],"format":"png","scale":2,"path":"C:\\designs\\out\\screen.png"}}}
```

### Auto layout

Set via `set_properties` with these props on a frame:

```json
{
  "layout": "horizontal",
  "gap": 12,
  "padding": 16,
  "mainAlign": "center",
  "crossAlign": "center",
  "sizingH": "hug",
  "sizingV": "fixed"
}
```

`sizingH`/`sizingV` values: `fixed`, `hug`, `fill` (children only).

### Components and instances

```json
// Create a component from selection
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"run_command","arguments":{"command":"component.create"}}}

// Place an instance
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"run_command","arguments":{"command":"instance.place","params":{"component":"<component id>","x":100,"y":200}}}}
```

### Inspect and verify

```json
// Check the document tree
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"inspect_document","arguments":{"depth":3}}}

// Screenshot to verify visually
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"screenshot","arguments":{"maxSize":1200}}}
```

## File Format Notes

- Files are `.figcraft` archives saved on local disk — no account needed.
- Can open `.fig` files (best-effort), `.svg`, and Figma REST API JSON.
- Export to `.fig` is experimental but supported.

## Troubleshooting

| Problem | Fix |
|---|---|
| `figcraft-cli` not found | Build with `cargo build --release -p figcraft-cli` or use AppImage on Linux |
| MCP not connecting | Check the path in `.mcp.json` is absolute |
| Text sizes wrong on CI | Run with `--no-system-fonts` for consistent layout |
| File access refused | Use `--automation-read-root` / `--automation-write-root` flags |
| Want to see edits live | Use `--attach 7979` with the app running on that port |

## Full Reference

For the complete command list (auto layout, vectors, prototypes, variables,
live sessions, branches, Dev Mode, plugins), read `figcraft-mcp.md` in this folder.
Only load it when the quick reference above is not enough, since it is large (162 KB).

- MCP reference: `docs/mcp.md` in the FigCraft repo
- CLI reference: `docs/cli.md`
- Agent guide: `AGENTS.md`
- Repo: https://github.com/storytold/figcraft