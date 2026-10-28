# FigCraft skill for AI agents

An agent skill that teaches coding agents how to design with
[FigCraft](https://github.com/storytold/figcraft), the open-source, offline, Figma-style design app,
through its built-in MCP server.

A skill and an MCP server are two separate pieces, and you need both:

| Piece | What it is | Where it lives |
|---|---|---|
| **Skill** (this repo) | Instructions: conventions, workflow, gotchas | `~/.agents/skills/figcraft/` |
| **MCP server** (ships with FigCraft) | The tools: `create_node`, `export`, `screenshot`… | your agent's MCP config |

Without the skill, the agent has the tools but guesses the conventions. Without the MCP server, the
agent knows what to do but can only fall back to the `figcraft-cli` command line.

## 1. Install FigCraft

Download an installer from the [FigCraft releases](https://github.com/storytold/figcraft/releases),
or build the CLI from source:

```
git clone https://github.com/storytold/figcraft
cd figcraft
cargo build --release -p figcraft-cli
```

The Windows installer puts `figcraft-cli.exe` in `C:\Program Files\FigCraft\`.

## 2. Install the skill

With the [`skills`](https://skills.sh/) CLI:

```
npx skills add <your-username>/figcraft-skill -g
```

Or copy `skills/figcraft/` into the skills folder your agent reads (for many agents that is
`~/.agents/skills/`):

```powershell
Copy-Item -Recurse skills\figcraft $HOME\.agents\skills\figcraft
```

Check it with `npx skills ls -g`.

## 3. Register the MCP server

The command is always `figcraft-cli mcp`. Only the place you register it differs per agent.

**Claude Code**

```
claude mcp add figcraft -- "C:\Program Files\FigCraft\figcraft-cli.exe" mcp
```

**Oh My Pi (OMP)**: run `/mcp add`, pick *User level*, then fill in:

```
Name:      figcraft
Transport: stdio
Command:   C:\Program Files\FigCraft\figcraft-cli.exe
Arguments: mcp
```

**Any agent that reads an `mcpServers` JSON file**

```json
{
  "mcpServers": {
    "figcraft": {
      "command": "C:\\Program Files\\FigCraft\\figcraft-cli.exe",
      "args": ["mcp"]
    }
  }
}
```

Config formats differ between agents (some use an array for `command`, Codex uses TOML), so check
your agent's MCP docs if this shape is rejected.

### Headless or live

- **Headless** (the config above): the agent works on an in-memory document. Nothing reaches disk
  until it saves or exports, so tell it where to save.
- **Live**: start the app with `figcraft --control 7979`, and register the server with
  `args: ["mcp", "--attach", "7979"]`. You watch every change in the app window, and several agents
  can share the same document.

To restrict where the agent can read and write files, add
`--automation-read-root <dir>` and `--automation-write-root <dir>` to the args.

## 4. Try it

> Use FigCraft to design a 375×812 mobile home screen with a greeting, a search field and a blue
> button. Save it to `C:\designs\home.figcraft` and export a PNG next to it.

## Updating the reference

`skills/figcraft/references/mcp.md` is a copy of FigCraft's `docs/mcp.md`. It changes as FigCraft
does. To refresh it:

```powershell
Invoke-WebRequest https://raw.githubusercontent.com/storytold/figcraft/HEAD/docs/mcp.md `
  -OutFile skills\figcraft\references\mcp.md
```

If you change the headings in that file, update the reference table at the bottom of `SKILL.md`.

## Credits and license

- `skills/figcraft/references/mcp.md` is copied from the FigCraft documentation, © 2026 ArtCraft
  Team and the FigCraft contributors, which is dual-licensed under MIT or Apache-2.0. See the
  upstream [`LICENSE-MIT`](https://github.com/storytold/figcraft/blob/main/LICENSE-MIT),
  [`LICENSE-APACHE`](https://github.com/storytold/figcraft/blob/main/LICENSE-APACHE), and
  [`NOTICE`](https://github.com/storytold/figcraft/blob/main/NOTICE).
- The ArtCraft name and logos are trademarks of the ArtCraft Team and are not part of this repo.
- The rest of this repo (`SKILL.md`, this README) is licensed under the license file you add here.
- This is an unofficial community skill and is not affiliated with the FigCraft project.
