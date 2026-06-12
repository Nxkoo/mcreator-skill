# MCreator Skill

[![skills.sh](https://skills.sh/b/Nxkoo/mcreator-skill)](https://skills.sh/Nxkoo/mcreator-skill)

An Agent Skill that helps Codex and other compatible AI coding agents work safely inside MCreator workspaces.

MCreator projects mix structured mod elements, generated code, locked elements, resources, plugins, and manual Java. This skill gives an AI agent explicit guidance for inspecting that structure before editing, preferring native MCreator capabilities, protecting code from regeneration, and using [MCreator Agent](https://github.com/Nxkoo/MCreator-Agent) MCP tools when the host agent exposes them.

The skill contains guidance and instructions only. It does not directly access files, edit workspaces, or call MCP tools by itself; those capabilities depend on the AI coding agent where it is installed.

## What It Covers

- MCreator workspace inspection before editing.
- MCreator-native-first implementation decisions.
- Lock-aware generated Java changes.
- Clean root-package and feature-package boundaries.
- Forge, NeoForge, Fabric, Bedrock, and datapack generator awareness.
- MCreator 2024.4 and Minecraft 1.20.1 / 1.21.1 workflows.
- GeckoLib asset, element, model, animation, and validation safety.
- Automatic detection and use of configured MCreator Agent MCP tools in Codex.
- Post-regeneration review and restoration of classes that should not have been deleted.
- Required validation and end-of-task reporting.

## Installation

Install interactively for supported agents:

```bash
npx skills add Nxkoo/mcreator-skill
```

Install specifically for Codex:

```bash
npx skills add Nxkoo/mcreator-skill -a codex
```

Install specifically for Cursor:

```bash
npx skills add Nxkoo/mcreator-skill -a cursor
```

Install specifically for Claude Code:

```bash
npx skills add Nxkoo/mcreator-skill -a claude-code
```

The repository exposes a valid root-level `SKILL.md`, so the public skills CLI discovers it without requiring full-depth scanning.

## Test Discovery

List the skills discoverable from this repository without installing:

```bash
npx skills add Nxkoo/mcreator-skill --list
```

## Usage

Invoke it explicitly:

```text
Use $mcreator-skill to inspect this MCreator workspace and fix the build error without refactoring unrelated files.
```

```text
Use $mcreator-skill to plan a GeckoLib entity safely and use MCreator Agent MCP tools when available to the host agent.
```

Codex may also invoke the skill automatically when a task matches the `SKILL.md` description.

## MCreator Agent MCP

The skill guides a compatible host agent to detect and use [MCreator Agent](https://github.com/Nxkoo/MCreator-Agent) tools already configured in Codex.

MCreator Agent must be running inside MCreator with a workspace open. Its local MCP endpoint normally starts at:

```text
http://localhost:5175/mcp
```

If port `5175` is busy, the active port is written to:

```text
%USERPROFILE%\.mcreator\mcp\port
```

Example Codex/VS Code MCP configuration:

```json
{
  "servers": {
    "mcreator-agent": {
      "type": "http",
      "url": "http://localhost:5175/mcp"
    }
  }
}
```

The guidance recommends configured MCP tools for workspace metadata, mod elements, locks, regeneration, builds, run targets, and GeckoLib operations. The host agent determines whether those tools and file inspection capabilities are available.

## Core Safety Rules

- Inspect the MCreator version, Minecraft version, generator, mod ID, root package, plugins, elements, locks, custom Java, and assets before editing.
- Prefer native mod element settings, procedures, resources, plugin features, and MCP tools before custom Java.
- Lock a generated mod element before directly editing its generated Java.
- Keep helpers, handlers, services, controllers, and feature systems out of generated technical packages such as `.entity`, `.item`, `.block`, `.procedures`, and `.init`.
- Read real GeckoLib assets and metadata before assuming model, animation, controller, or texture names.
- Before regeneration, capture the current state. After regeneration, evaluate every deleted Java class and restore project-owned code that should not have been removed.

See [SKILL.md](./SKILL.md) for the complete workflow and policies.

## Repository Structure

```text
mcreator-skill/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── LICENSE
└── README.md
```

## License

MIT
