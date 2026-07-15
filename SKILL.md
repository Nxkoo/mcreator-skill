---
name: mcreator-skill
description: Use when AI coding agents work with MCreator workspaces, mod elements, GeckoLib workflows, .mcreator metadata.files tracking, generated-code boundaries, and safe validation loops.
---

# MCreator Skill

Use this skill as guidance for working safely inside MCreator workspaces. The host AI coding agent remains responsible for file access, tool use, MCP calls, edits, and validation. Prioritize native MCreator behavior, preserve regeneration safety, keep Java packages clean, and guide the host agent to use MCreator Agent tools when they are available.

## MCP-Aware Guidance

When the host agent is Codex, guide it to check whether the MCreator Agent MCP server is already configured for the current session before asking the user.

The host agent should inspect its active tool/MCP surface for an MCreator Agent server or tools/resources matching `mcreator-agent`, `getWorkspaceInfo`, `listModElements`, `setModElementLock`, `regenerateCode`, `buildWorkspace`, `getGeckoLibStatus`, `listGeckoLibAssets`, `importGeckoLibAssets`, `createGeckoLibElement`, or `validateGeckoLibElement`.

If those tools are available to the host agent, prefer them for supported workspace operations before falling back to manual file inspection or shell commands. Prefer MCP for workspace metadata, mod element listing, lock state, element creation/deletion, lock/unlock, regeneration, builds, client/server runs, plugin state, and GeckoLib asset/element validation.

If Codex does not expose the tools directly, guide the host agent to check whether MCreator Agent is reachable only when local HTTP access is appropriate for the environment:

- MCreator Agent usually serves MCP at `http://localhost:5175/mcp`, or the next free port.
- The active port is written to `%USERPROFILE%\.mcreator\mcp\port`.
- A basic MCP readiness probe is a JSON-RPC `tools/list` POST to the MCP endpoint.

MCreator must be running with the plugin loaded and a workspace open. If MCP is configured but unavailable, the host agent should report that state and continue with safe read-only inspection unless the user asks to repair the MCP setup.

## Core Workflow

1. Inspect before editing.
2. Prefer MCreator-native configuration before custom code.
3. Classify every file that may be edited.
4. Lock generated mod elements before direct generated-Java edits.
5. Register created/imported/edited element-owned files in `.mcreator` `metadata.files` before regeneration or build checks.
6. Keep custom Java under the mod root package with clean feature boundaries.
7. Validate with the narrowest reliable build/check path.
8. Report locks, custom Java, assets, metadata files, validation, and remaining manual MCreator testing.

## Workspace Inspection

Before changing files, inspect or infer:

- MCreator version.
- Minecraft version.
- Generator/loader, such as Forge, NeoForge, Fabric, Bedrock, or datapack.
- Mod ID and root Java package.
- Installed plugins, especially GeckoLib.
- Existing mod elements and their types.
- Existing locked elements.
- Existing `.mcreator` `metadata.files` entries for affected elements.
- Existing custom Java.
- Existing assets, models, textures, animations, lang files, tags, recipes, and loot tables.
- Whether MCreator Agent/MCP tools are available.

Use the workspace files and available tools as the source of truth. Do not assume version, generator, plugin, or asset state from the user's prompt alone.

## Workspace State Consistency

A mod element is not fully created or configured until all available validation layers agree:

- `elements/<Name>.mod.json` existing is not enough. When applicable, the `.mcreator` workspace file must also include the element in `mod_elements`.
- When MCP is available, `listModElements` must list the element in the currently open MCreator workspace.
- For GeckoLib elements, `validateGeckoLibElement` must see the element when that tool is available.
- If filesystem and MCP state disagree, report the workspace as stale/divergent.
- Do not claim an element was fully created in MCreator when `listModElements` does not show it. Only claim that a filesystem fallback was performed, and state that the MCreator workspace/UI may need reload or refresh.

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
- Do not assume `listGeckoLibAssets` means the runtime renderer can load an asset. Verify both authoring/import paths and runtime resource paths.
- Check Java `GeoModel` and renderer resource locations, then confirm files exist at those exact paths.
- For GeckoLib entities/items, validate likely runtime paths such as `src/main/resources/assets/<modid>/geo/<name>.geo.json`, `src/main/resources/assets/<modid>/animations/<name>.animation.json`, `src/main/resources/assets/<modid>/textures/entities/<name>.png`, and applicable item textures/models.
- If paths differ by plugin, generator, or version, inspect the actual generated Java and resource tree instead of assuming.

When MCreator Agent/MCP GeckoLib tools exist, prefer the GeckoLib-specific path for animated element creation/validation rather than a generic element creation path.

### Item/Blockbench Model Texture References

When importing Blockbench item models, inspect internal texture references before declaring success. Fix wrong namespaces, folders, non-ASCII names, unrelated texture names, or references such as `block/...` when the texture belongs under `textures/item/...`. Do not only copy the model JSON; verify the texture references inside it.

## MCP And MCreator Agent

When MCreator Agent/MCP tools are available to the host agent, guide it to prefer the configured Codex MCP tools directly. Discover the actual tool names first and use the available equivalents for:

- Listing workspace metadata.
- Listing mod elements and locked elements.
- Creating or modifying mod elements.
- Deleting mod elements when requested.
- Importing assets.
- Locking or unlocking mod elements.
- Generating a single element (prefer over full regen).
- Regenerating code (dangerous; full workspace).
- Running builds or client/server checks.
- Reading plugin state.
- Validating GeckoLib setup.

Do not invent exact MCP tool names. Prefer tools after confirming they are listed by the active MCP session.

Useful MCreator Agent resources may include `workspace://overview`, `workspace://elements`, `workspace://structure`, `workspace://geckolib/status`, and `workspace://geckolib/assets`. Read them when available instead of rediscovering the same facts from disk.

### MCP tool truth table

| Tool | Creates element | Generates Java | Scope | Notes |
|---|---|---|---|---|
| `importGeckoLibAssets` | no | no | assets only | Prefer for geo/anim/texture import |
| `createGeckoLibElement` | yes (scaffold) | only if `generateCode=true` and tool supports it | one element metadata | **Not done** until Java/registries exist |
| `updateGeckoLibElement` | updates definition | no | one element | Prefer over hand-editing `.mod.json` |
| `generateModElement` | no | yes | one element (+ base registries) | Prefer over full `regenerateCode` |
| `regenerateCode` | no | full workspace | **dangerous** | May delete untracked Java; rewrite `mcreator.gradle` |
| `buildWorkspace` | no | no (build) | workspace | Await completion report when available |
| `validateGeckoLibElement` | no | no | one element | Check assets/codegen postconditions |

### Hard rules (agents)

1. Treat `createGeckoLibElement` success **without** generated entity/model/renderer/init as **incomplete**.
2. Prefer `generateModElement` (or create with `generateCode=true`) over full `regenerateCode` after creating GeckoLib entities.
3. Do **not** hand-edit `elements/*.mod.json` when `updateGeckoLibElement` exists — disk edits diverge from the open MCreator in-memory model.
4. Do **not** call full `regenerateCode` after create unless single-element generate is unavailable and a pre-regen snapshot exists (`git status`/file inventory). Always inspect deleted Java and `mcreator.gradle` afterward.
5. Never trust fire-and-forget success strings like "initiated successfully" as proof of completion; require completion status + deleted/modified file report when the tool provides them.
6. Register `metadata.files` before any regen (see Metadata Files Policy).
7. If `mcreator.gradle` loses custom deps (e.g. `flatDir` / local jars), restore them before claiming build success.

### Preferred GeckoLib entity flow

```text
getGeckoLibStatus
→ importGeckoLibAssets
→ createGeckoLibElement (complete definition: model, texture, hitbox, animations, headMovement)
→ generateModElement  OR  create(... generateCode:true)
→ validateGeckoLibElement
→ compileJava / buildWorkspace (await completion)
→ lock only if customizing generated Java
```

### Head movement

`headMovement=true` in definition is not enough by itself for custom bone layouts. After generation, verify the model class rotates the correct bone name(s). If `nose`/`headwear` are siblings of `head` (not children), multi-bone rotation may be required.

MCreator Agent cautions:

- MCP-created elements must persist both the in-memory workspace state and the on-disk `.mod.json`.
- After create/delete or other workspace mutations, the MCreator UI may need an explicit workspace panel refresh.
- Helper/client/network files that must survive regeneration must be represented in the owning mod element's `.mcreator` `metadata.files`.
- Broad local Gradle or MCreator tests can be polluted by unrelated plugins in `%USERPROFILE%\.mcreator\plugins`; prefer the narrow reliable validation target for the current repo and report environmental pollution separately.
- Full regenerate historically wiped custom `mcreator.gradle` dependency blocks and untracked package classes; always diff after regen.

## MCreator Metadata Files Policy

When the host agent creates, imports, edits, or manually adds any file that belongs to a mod element, it must register that file in the workspace `.mcreator` file under the owning mod element's `metadata.files` before running regeneration or build checks.

This is mandatory for regeneration safety. MCreator may delete, ignore, or fail to track files that are not represented in element metadata.

Apply this policy whether the host agent uses MCreator MCP tools, MCreator Agent tools, GeckoLib tools, manual file edits, or fallback Java/resource creation.

For each affected mod element, guide the host agent to:

- Ensure the element entry in `<workspace>.mcreator` has a `metadata` object.
- Ensure `metadata.files` includes every Java, resource, model, texture, animation, lang, tag, recipe, loot table, helper, renderer, network, or client file owned by that element.
- Include generated entity, item, block, procedure, init, renderer, model, menu, network, and client classes when they are owned by or required for that mod element.
- Include runtime resource paths, not only workspace/import paths.
- Include manually created fallback files before running regeneration or build checks.
- Re-read the `.mcreator` file after mutations and verify the files are listed.
- Avoid relying only on `.mod.json`; `.mod.json` defines the element, while `.mcreator` tracks files that must survive workspace operations.

For GeckoLib elements, include both MCreator import paths and runtime resource paths when both exist, for example:

```text
models/<name>.geo.json
models/animations/<name>.animation.json
src/main/resources/assets/<modid>/geo/<name>.geo.json
src/main/resources/assets/<modid>/animations/<name>.animation.json
src/main/resources/assets/<modid>/textures/entities/<name>.png
src/main/java/<basepackage>/entity/<Name>Entity.java
src/main/java/<basepackage>/client/model/<Name>Model.java
src/main/java/<basepackage>/client/renderer/<Name>Renderer.java
```

## Regeneration Safety

Before regeneration or builds:

- Do not run `regenerateCode` when the currently open MCreator/MCP workspace does not recognize the mod element just created or modified. Regeneration in this divergent state may delete or overwrite manual Java, classes, or assets created as fallback. Refresh, reload, or reconcile workspace state first, or explicitly report that regeneration was skipped for safety.
- Confirm whether edits live in files MCreator may overwrite.
- Prefer native element metadata/resources for generated behavior.
- Verify all created/imported/edited element-owned files are listed in the owning element's `.mcreator` `metadata.files`.
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

### False Success Red Flags

- Gradle builds, but `listModElements` does not show the element.
- A `.mod.json` exists, but MCP validation fails.
- The GeckoLib plugin is loaded, but `typesAvailable=false`.
- Assets are listed, but Java resource locations point somewhere else.
- Model JSON exists, but internal texture references point to wrong folders or assets.
- Regeneration would run while the open MCreator workspace is stale.

## Task Report

End every task with a validation matrix that distinguishes:

- Filesystem changes: files written or edited.
- MCreator/MCP recognition: `listModElements`.
- GeckoLib element validation: `validateGeckoLibElement`.
- GeckoLib asset discovery: `listGeckoLibAssets`.
- Java/resource compilation: Gradle or `buildWorkspace`.
- Manual runtime/UI validation still needed: inventory model, texture, spawn, hitbox, tooltip, animation, and behavior.

Also report which mod elements and assets changed, which files were added to `.mcreator` `metadata.files`, whether elements were locked, whether custom Java was used and why, and any regeneration deletions/restorations. Never collapse validation layers into a single "done" claim. Say exactly which layers passed, failed, were unavailable, or were skipped.
