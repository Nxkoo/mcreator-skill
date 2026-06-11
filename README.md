# MCreator AI Skill

An Agent Skill that helps Codex and other compatible AI coding agents work safely inside MCreator workspaces.

MCreator projects mix structured mod elements, generated code, locked elements, resources, plugins, and manual Java. This skill gives an AI agent explicit rules for inspecting that structure before editing, preferring native MCreator capabilities, protecting code from regeneration, and using [MCreator Agent](https://github.com/Nxkoo/MCreator-Agent) MCP tools when available.

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

### Codex user skill

Clone the repository into your user skills directory:

```powershell
git clone https://github.com/Nxkoo/mcreator-skill.git "$env:USERPROFILE\.agents\skills\mcreator-ai"
```

Codex detects skills from `%USERPROFILE%\.agents\skills`. Restart Codex if the skill does not appear immediately.

To update later:

```powershell
git -C "$env:USERPROFILE\.agents\skills\mcreator-ai" pull
```

### Repository-scoped skill

To make the skill available only inside a specific repository:

```powershell
git clone https://github.com/Nxkoo/mcreator-skill.git ".agents\skills\mcreator-ai"
```

Codex scans `.agents/skills` from the current working directory up to the repository root.

### Other Agent Skills-compatible tools

This repository follows the Agent Skills format: a root `SKILL.md` with optional `agents/openai.yaml` metadata. Install or import this repository using your agent's supported skill installation flow.

## Usage

Invoke it explicitly:

```text
Use $mcreator-ai to inspect this MCreator workspace and fix the build error without refactoring unrelated files.
```

```text
Use $mcreator-ai to create a GeckoLib entity safely and use MCreator Agent MCP tools when available.
```

Codex may also invoke the skill automatically when a task matches the `SKILL.md` description.

## MCreator Agent MCP

The skill can automatically detect and use [MCreator Agent](https://github.com/Nxkoo/MCreator-Agent) tools already configured in Codex.

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

The skill prefers configured MCP tools for workspace metadata, mod elements, locks, regeneration, builds, run targets, and GeckoLib operations. It falls back to safe file inspection when the MCP server is unavailable.

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
