---
name: mcreator-ai
description: Safe MCreator workspace inspection, planning, editing, and review for Minecraft mod development. Use when working in MCreator workspaces or with MCreator 2024.4, Minecraft 1.20.1 or 1.21.1, Forge, NeoForge, Fabric, Bedrock/datapack generators, GeckoLib plugins, MCreator Agent/MCP tools, mod element creation or editing, build fixes, lock decisions, custom Java, package hygiene, assets, resources, recipes, tags, loot tables, localization, or workspace regeneration safety.
---

# MCreator AI

Use this skill to work safely inside MCreator workspaces. Prioritize native MCreator behavior, preserve regeneration safety, keep Java packages clean, and use MCP/MCreator Agent tools when they are available.

## Codex MCP Detection

When running in Codex, assume the MCreator Agent MCP server may already be configured for the current session. Do not ask the user whether it is configured before checking.

First, inspect the active Codex tool/MCP surface for an MCreator Agent server or tools/resources matching `mcreator-agent`, `getWorkspaceInfo`, `listModElements`, `setModElementLock`, `regenerateCode`, `buildWorkspace`, `getGeckoLibStatus`, `listGeckoLibAssets`, `importGeckoLibAssets`, `createGeckoLibElement`, or `validateGeckoLibElement`.

If those tools are available, use them for supported workspace operations before falling back to manual file inspection or shell commands. Prefer MCP for workspace metadata, mod element listing, lock state, element creation/deletion, lock/unlock, regeneration, builds, client/server runs, plugin state, and GeckoLib asset/element validation.

If Codex does not expose the tools directly, check whether MCreator Agent is reachable only when local HTTP access is appropriate for the environment:

- MCreator Agent usually serves MCP at `http://localhost:5175/mcp`, or the next free port.
- The active port is written to `%USERPROFILE%\.mcreator\mcp\port`.
- A basic MCP readiness probe is a JSON-RPC `tools/list` POST to the MCP endpoint.

MCreator must be running with the plugin loaded and a workspace open. If MCP is configured but unavailable, report that state and continue with safe read-only inspection unless the user asks to repair the MCP setup.

## Core Workflow

1. Inspect before editing.
2. Prefer MCreator-native configuration before custom code.
3. Classify every file that may be edited.
4. Lock generated mod elements before direct generated-Java edits.
5. Keep custom Java under the mod root package with clean feature boundaries.
6. Validate with the narrowest reliable build/check path.
7. Report locks, custom Java, assets, validation, and remaining manual MCreator testing.

## Workspace Inspection

Before changing files, inspect or infer:

- MCreator version.
- Minecraft version.
- Generator/loader, such as Forge, NeoForge, Fabric, Bedrock, or datapack.
- Mod ID and root Java package.
- Installed plugins, especially GeckoLib.
- Existing mod elements and their types.
- Existing locked elements.
- Existing custom Java.
- Existing assets, models, textures, animations, lang files, tags, recipes, and loot tables.
- Whether MCreator Agent/MCP tools are available.

Use the workspace files and available tools as the source of truth. Do not assume version, generator, plugin, or asset state from the user's prompt alone.

## Editing Priority

Prefer solutions in this order:

1. Native MCreator mod element configuration.
2. MCreator procedures.
3. Assets, resources, tags, recipes, loot tables, and localization.
4. Plugin features, especially GeckoLib plugin features.
5. MCP or MCreator Agent tools, when available.
6. Custom Java only when necessary.

Do not edit Gradle, generator settings, mod ID, namespace, Minecraft version, or plugin setup unless the user explicitly asks.

## File Classification

Before creating or editing Java/resources, classify each affected file as one of:

- Generated mod element class.
- Locked mod element class.
- Generated registry/init file.
- Procedure file.
- Renderer/model class.
- Manual feature class.
- Asset/resource file.

If unsure, inspect further and avoid destructive edits. Treat anything MCreator can regenerate as unsafe until its ownership is clear.

## Lock Policy

- If direct edits are needed in generated Java for a mod element, lock that mod element first using MCreator/MCP or the available equivalent.
- Do not lock when behavior can be done with procedures, mod element settings, resources, or plugin features.
- Locking is appropriate for custom tick logic, interaction behavior, travel/movement, projectile impact, damage handling, AI goals, hitbox behavior, render behavior, or low-level events not exposed by MCreator.
- After locking, edit the generated class only as needed and report that the element is now manually maintained.
- For base/main files, prefer user code blocks when MCreator provides them.

## Package Architecture

Keep all Java code under the workspace root Java package. Do not invent external `custom` packages outside the mod root package.

Keep MCreator technical packages semantically clean:

- `.entity` contains entity classes only.
- `.item` contains item classes only.
- `.block` contains block classes only.
- `.procedures` contains procedure classes only.
- `.init` contains registry/init classes only.
- `.client.model` contains model classes only.
- `.client.renderer` contains renderer classes only.

Do not place helpers, constants, handlers, services, controllers, managers, or feature systems in those technical packages.

For feature-specific custom code, create a root-level feature package using a registry-safe feature name:

```text
dev.nykoo.mymcreatormod.entity.VillagerTankEntity
dev.nykoo.mymcreatormod.entity.VillagerTankProjectileEntity
dev.nykoo.mymcreatormod.villagertank.VillagerTankConstants
dev.nykoo.mymcreatormod.villagertank.VillagerTankControls
dev.nykoo.mymcreatormod.villagertank.VillagerTankInputHandler
```

Do not use `.entity` as a dumping ground for feature logic.

## Entity Editing

- Lock generated entity mod elements before direct custom behavior edits.
- Keep locked entity classes focused on entity behavior and integration.
- Move constants, handlers, controllers, and shared systems to the feature package.
- Let locked generated classes call feature-package code rather than accumulating all feature logic inline.

## GeckoLib

GeckoLib is a first-class concern for this skill.

- Do not assume animation, controller, model, texture, or renderer names. Read `.geo`, `.animation`, `.png`, renderer/model classes, and related files first.
- Keep asset names consistent across element metadata, JSON, textures, models, animations, lang, and generated code.
- Import GeckoLib assets through MCP/tools when available.
- Separate visual model/animation concerns from gameplay logic.
- Do not make hitbox or gameplay logic depend on animation timing unless the user explicitly requires it.
- Lock GeckoLib-generated entity/item/block/armor elements before direct generated Java edits.
- Temporary GeckoLib entities must clean themselves up safely with timers or equivalent logic.

When MCreator Agent/MCP GeckoLib tools exist, prefer the GeckoLib-specific path for animated element creation/validation rather than a generic element creation path.

## MCP And MCreator Agent

When MCreator Agent/MCP tools are available, prefer the configured Codex MCP tools directly. Discover the actual tool names first and use the available equivalents for:

- Listing workspace metadata.
- Listing mod elements and locked elements.
- Creating or modifying mod elements.
- Deleting mod elements when requested.
- Importing assets.
- Locking or unlocking mod elements.
- Regenerating code.
- Running builds or client/server checks.
- Reading plugin state.
- Validating GeckoLib setup.

Do not invent exact MCP tool names. If the current Codex session exposes tools such as `getWorkspaceInfo`, `listModElements`, `createElement`, `deleteElement`, `setModElementLock`, `regenerateCode`, `buildWorkspace`, `runClient`, `runServer`, `getGeckoLibStatus`, `listGeckoLibAssets`, `importGeckoLibAssets`, `createGeckoLibElement`, or `validateGeckoLibElement`, prefer them for their matching operation after confirming they are currently available.

Useful MCreator Agent resources may include `workspace://overview`, `workspace://elements`, `workspace://structure`, `workspace://geckolib/status`, and `workspace://geckolib/assets`. Read them when available instead of rediscovering the same facts from disk.

MCreator Agent cautions:

- MCP-created elements must persist both the in-memory workspace state and the on-disk `.mod.json`.
- After create/delete or other workspace mutations, the MCreator UI may need an explicit workspace panel refresh.
- Helper/client/network files that must survive regeneration may need to be represented in element metadata.
- Broad local Gradle or MCreator tests can be polluted by unrelated plugins in `%USERPROFILE%\.mcreator\plugins`; prefer the narrow reliable validation target for the current repo and report environmental pollution separately.

## Regeneration Safety

Before regeneration or builds:

- Confirm whether edits live in files MCreator may overwrite.
- Prefer native element metadata/resources for generated behavior.
- Keep manual feature classes outside generated technical packages.
- Avoid formatting or rewriting generated files unless necessary for the requested change.
- Capture the pre-regeneration state using version-control status/diff or an equivalent file inventory so deleted classes can be identified afterward.

After running MCP `regenerateCode` or any equivalent MCreator regeneration:

- Inspect the resulting diff and evaluate every deleted Java class.
- Determine whether each deletion is expected generated-code cleanup or an unintended removal.
- Do not blindly restore every deleted class; leave obsolete generated output deleted when regeneration intentionally replaced or removed it.
- Restore a deleted class when it is a locked mod element class, manual feature class, required helper/controller/handler/service, or other project-owned code that regeneration should not remove.
- Restore from version control, the pre-regeneration snapshot, or another verified source without overwriting unrelated post-regeneration changes.
- Investigate why an unintended deletion happened. Move manual code to a safe root-level feature package or represent required generated companions in element metadata when that is necessary for them to survive future regeneration.
- Re-run the appropriate validation after restoration.
- Report every class that regeneration deleted, which classes were restored or intentionally left deleted, and the reason for each decision.

## Task Report

End every task with:

- What was added or changed.
- Which mod elements were created or edited.
- Which assets were imported or edited.
- Which files were changed.
- Whether any mod element was locked.
- Whether custom Java was used.
- Why custom Java was necessary, if used.
- What validation was run.
- What still needs manual testing inside MCreator.
