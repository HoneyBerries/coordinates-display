# Merge BoxLib Into CoordinatesDisplay++ and Port to Minecraft 26.3 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship CoordinatesDisplay++ on Minecraft 26.3 with BoxLib's source living inside this repo, so the mod no longer needs the separate BoxLib mod installed.

**Architecture:** Port the mod to 26.3 first against the published BoxLib `23.0.0-snapshot` jar (isolates MC breakage from the merge). Then copy BoxLib's source verbatim into the existing `common`/`fabric`/`neoforge` modules **keeping the `dev.boxadactle.boxlib` package name and deleting nothing**, wire its entrypoints and mixin configs into this mod's own entrypoints, and drop the external dependency. Keeping package names means upstream's `import dev.boxadactle.boxlib...` lines never conflict when syncing from upstream.

**Tech Stack:** Gradle (NeoForge moddev 2.0.148, Fabric Loom), Java 25, Mixin 0.8.5, Fabric API 0.161.0+26.3, NeoForge 26.3.0.70-beta.

**Spec:** Conversation of 2026-11-07 (no separate spec doc). Decisions: work on `main` of `~/Desktop/Projects/Minecraft/CoordinatesDisplay++`; copy from `~/Desktop/Software/BoxLib` @ `4ee4945`; copy all BoxLib code, no deletions; keep upstream-syncability.

## Global Constraints

- Minecraft `26.3`, `neo_form_version=26.3-1`, `fabric_version=0.161.0+26.3`, `fabric_loader_version=0.19.5`, `modmenu_version=21.0.0`, `neoforge_version=26.3.0.70-beta` (values copied from BoxLib `gradle.properties`).
- Java 25. Platforms: `fabric,neoforge` only (no `forge` module exists).
- BoxLib source keeps package `dev.boxadactle.boxlib` and mod-id constant `boxlib`; BoxLib source files are copied unmodified except where a task below names the edit.
- Mod id stays `coordinatesdisplay` (keeps config files, keybinds and upstream merges stable).
- Do not touch `~/Desktop/Software/BoxLib` (read-only source).
- Fork's `origin` is `HoneyBerries/coordinates-display`; add `upstream` remote (original repo URL, supplied by the user) before any sync.

## Review Focus

- Standalone `boxlib` mod installed next to this mod: should produce a clear "incompatible mod" message, not a duplicate-class crash (Task 5 asserts the metadata).
- BoxLib dev-only test code (`TestMod`, `ExampleConfigScreen`, title-screen "Open Config" button) must not run in production, and in dev must not replace this mod's own config screen (Tasks 3, 4).
- `Scheduling.tick()` must run exactly once per client tick, not twice and not never (Tasks 3, 4 runtime check).
- Client commands (`/coordinates ...`) must register and execute on both loaders; they depend on the BoxLib command-source mixins (Tasks 3, 4).
- Existing users' config file location/name must be unchanged after the merge (Task 6).

---

### Task 1: Port to 26.3 against published BoxLib 23.0.0-snapshot

**Files:**
- Modify: `gradle.properties`, `build.gradle` (moddev plugin `2.0.148`)
- Modify: whatever fails to compile in `common/`, `fabric/`, `neoforge/`

**Interfaces:**
- Produces: a repo that builds and runs on 26.3 with `boxlib_version=23.0.0-snapshot`; all later tasks start from it.

- [ ] **Step 1: Record baseline.** Run `./gradlew build` on the untouched tree. Expected: BUILD SUCCESSFUL (if not, stop and report; the baseline is broken).
- [ ] **Step 2: Bump `gradle.properties`** to the Global Constraints versions; set `minecraft_version=26.3`, `minecraft_version_range=[26,27)`, `minecraft_version_range_fabric=>=26.3`, `game_versions = 26.3`, `enabled_platforms=fabric,neoforge`, `boxlib_version=23.0.0-snapshot`, `boxlib_version_range = [23, 24)`, `boxlib_version_range_fabric=23.x`. Bump moddev plugin in `build.gradle` to `2.0.148`.
- [ ] **Step 3: Run `./gradlew build`.** Fix compile errors in this mod's own code only, using `git -C ~/Desktop/Software/BoxLib show 97c5ab7 7d4b7f3 5a8186c` as the reference for what the 26.2→26.3 API changes look like. If `23.0.0-snapshot` does not resolve from `maven.boxadactle.dev`, skip to Task 2 and do the 26.3 fixes there.
- [ ] **Step 4: Run `./gradlew :fabric:runClient`, then `:neoforge:runClient`.** Expected: reaches title screen, enter a world, HUD renders, `/coordinates` command exists.
- [ ] **Step 5: Commit** `git commit -am "Port to Minecraft 26.3"`

### Task 2: Vendor BoxLib `common` source and resources

**Files:**
- Create: `common/src/main/java/dev/boxadactle/boxlib/**` (copy of BoxLib `common/src/main/java/dev/boxadactle/boxlib/**`, all packages including `test`, `http`, `function`)
- Create: `common/src/main/resources/boxlib-common.mixins.json`, `common/src/main/resources/assets/boxlib/lang/en_us.json` (copied verbatim)
- Modify: `common/build.gradle` (delete the `compileOnly "dev.boxadactle:Boxlib-common:..."` line)
- Create: `docs/boxlib-merge.md` recording the BoxLib commit copied (`4ee4945`) and the sync procedure from Task 6.

**Interfaces:**
- Consumes: Task 1's building repo.
- Produces: `dev.boxadactle.boxlib.*` classes compiled inside `:common` (same FQCNs `fabric`/`neoforge` and this mod already import). `boxlib-common.mixins.json` lists `DebugRendererMixin`, `KeybindAccessor` (not registered in loader metadata until Tasks 3/4). Do **not** register BoxLib's common `TitleScreenMixin` (BoxLib itself omits it).

- [ ] **Step 1: Copy** with `cp -r ~/Desktop/Software/BoxLib/common/src/main/java/dev/boxadactle/boxlib common/src/main/java/dev/boxadactle/` and the two resource files; do not copy BoxLib's `icon.png` (this repo has its own).
- [ ] **Step 2: Remove the `compileOnly` BoxLib line** from `common/build.gradle`.
- [ ] **Step 3: Run `./gradlew :common:compileJava`.** Expected: BUILD SUCCESSFUL, no duplicate-class errors.
- [ ] **Step 4: Commit** `git add -A && git commit -m "Vendor BoxLib common source (BoxLib@4ee4945)"`

### Task 3: Fabric entrypoints, mixins and dependency removal

**Files:**
- Create: `fabric/src/main/java/dev/boxadactle/boxlib/fabric/BoxLibFabric.java` and `.../mixin/FabricClientCommandSourceMixin.java`, `.../mixin/TitleScreenMixin.java` (copied from BoxLib `fabric`); also copy `BoxLibModmenu.java` but never register it.
- Create: `fabric/src/main/resources/boxlib.mixins.json` (copied)
- Modify: `fabric/src/main/resources/fabric.mod.json`, `fabric/build.gradle`

**Interfaces:**
- Consumes: Task 2 classes `BoxLib.init()`, `BCommandImpl.register(CommandDispatcher<BCommandSourceStack>)`, `Scheduling.tick()`, `ModConstants.IS_DEVELOPMENT`, `TestMod.init()`.
- Produces: BoxLib initialized on Fabric via a second `client` entrypoint beside this mod's `main` entrypoint.

- [ ] **Step 1: Copy** the three Java files plus `BoxLibModmenu.java` and `boxlib.mixins.json`. Edit `boxlib.mixins.json` to remove `TitleScreenMixin` from `"mixins"` (dev-only button; keep the source file).
- [ ] **Step 2: Edit `fabric.mod.json`:** add `"client": ["dev.boxadactle.boxlib.fabric.BoxLibFabric"]` to `entrypoints`; add `"boxlib.mixins.json"` and `"boxlib-common.mixins.json"` to `mixins`; remove the `"boxlib"` entry from `depends`; add `"breaks": { "boxlib": "*" }`. Do not add `BoxLibModmenu` to `modmenu` entrypoints.
- [ ] **Step 3: Edit `fabric/build.gradle`:** delete the `Boxlib-fabric` implementation line; extend the `processResources` refmap hack so it also patches `boxlib-common.mixins.json` and `boxlib.mixins.json` (same `${mod_id}.refmap.json` value), mirroring BoxLib's own `fabric/build.gradle`.
- [ ] **Step 4: Run `./gradlew :fabric:build`.** Expected: BUILD SUCCESSFUL.
- [ ] **Step 5: Run `./gradlew :fabric:runClient`.** Expected: title screen loads without mixin errors in the log; in a world, `/coordinates` works; a `Scheduling`-driven feature fires once (check Bindings behavior that uses `Scheduling`); no "Open Config" button on title screen.
- [ ] **Step 6: Commit** `git add -A && git commit -m "Wire vendored BoxLib into Fabric"`

### Task 4: NeoForge entrypoints, mixins and dependency removal

**Files:**
- Create: `neoforge/src/main/java/dev/boxadactle/boxlib/neoforge/BoxLibNeoforge.java`, `.../mixin/CommandSourceStackMixin.java`, `neoforge/src/main/resources/boxlib.mixins.json` (copied)
- Modify: `neoforge/src/main/java/dev/boxadactle/coordinatesdisplay/neoforge/CoordinatesDisplayNeoforge.java`, `neoforge/src/main/resources/META-INF/neoforge.mods.toml`, `neoforge/build.gradle`

**Interfaces:**
- Consumes: Task 2 classes (same as Task 3) plus `MouseUtils.setMouseDown(int)` / `setMouseUp()`.
- Produces: BoxLib initialized on NeoForge under this mod's id.

- [ ] **Step 1: Copy** the three files. Edit the copied `BoxLibNeoforge`: remove the `@Mod` annotation (this mod already owns the `@Mod`), change `@EventBusSubscriber(modid = ModConstants.MOD_ID, ...)` to `modid = CoordinatesDisplay.MOD_ID`, and delete the `registerExtensionPoint(IConfigScreenFactory...)` call in `load()` so dev runs keep this mod's config screen.
- [ ] **Step 2: In `CoordinatesDisplayNeoforge` constructor,** replicate the dropped constructor body: set `ModConstants.IS_DEVELOPMENT = !FMLLoader.getCurrent().isProduction()` then `BoxLib.init()` (before the existing `registerExtensionPoint`).
- [ ] **Step 3: Edit `neoforge.mods.toml`:** delete the `boxlib` required dependency block; add `[[mixins]] config = "boxlib.mixins.json"` and `[[mixins]] config = "boxlib-common.mixins.json"`; add a dependency block with `modId = "boxlib"`, `type = "incompatible"`, `versionRange = "*"`, `ordering = "NONE"`, `side = "BOTH"`, `reason = "CoordinatesDisplay++ bundles BoxLib; remove the standalone BoxLib mod."`.
- [ ] **Step 4: Edit `neoforge/build.gradle`:** delete `implementation "dev.boxadactle:Boxlib-neoforge:..."` (the whole `dependencies` block).
- [ ] **Step 5: Run `./gradlew :neoforge:build`, then `:neoforge:runClient`.** Expected: BUILD SUCCESSFUL; same in-game checks as Task 3 Step 5.
- [ ] **Step 6: Commit** `git add -A && git commit -m "Wire vendored BoxLib into NeoForge"`

### Task 5: Remove leftover BoxLib dependency plumbing and document

**Files:**
- Modify: `gradle.properties`, `buildSrc/src/main/groovy/multiloader-common.gradle`, `build.gradle`, `README.md`, `CHANGELOG.md`

**Interfaces:**
- Consumes: Tasks 3-4 (templates no longer reference `boxlib_version_range*`).

- [ ] **Step 1: Delete** the `boxlib_version*` lines from `gradle.properties`, the two `boxlib_version_range*` entries from `expandProps` in `multiloader-common.gradle`, and the two `maven.boxadactle.dev` repository blocks in `build.gradle` and `multiloader-common.gradle` publishing section only if they are no longer needed for anything else (leave the publishing target if it is this mod's own publish repo).
- [ ] **Step 2: Run `./gradlew clean build`.** Expected: BUILD SUCCESSFUL for fabric and neoforge. Then assert the metadata: `unzip -p fabric/build/libs/*.jar fabric.mod.json | grep -c boxlib` shows the `breaks` entry and no `depends` entry; same for `neoforge.mods.toml` (`type = "incompatible"`).
- [ ] **Step 3: Update `README.md`** (remove both "requires BoxLib" notices; add "BoxLib is bundled; remove the standalone BoxLib mod") and add a `CHANGELOG.md` entry for 26.3 + bundled BoxLib.
- [ ] **Step 4: Commit** `git commit -am "Drop external BoxLib dependency, update docs"`

### Task 6: Full verification and sync setup

**Files:**
- Modify: `docs/boxlib-merge.md`

- [ ] **Step 1: Config location check.** Before/after comparison: with the Task 1 commit, run `:fabric:runClient`, note the config file under `runs/client/config/` (BConfigHandler path); on the final commit confirm the same filename is produced and an existing config loads (no reset to defaults).
- [ ] **Step 2: Manual test matrix** on both loaders: HUD renders, config screen opens (Mod Menu on Fabric; mods list on NeoForge), keybinds work, `/coordinates` runs, mark position works, death position shows.
- [ ] **Step 3: Sync setup.** `git remote add upstream <original repo URL>`; `git fetch upstream`; record in `docs/boxlib-merge.md` the procedure: `git merge upstream/dev` (merge, not rebase, since `main` is pushed); on conflicts in `gradle.properties`/`fabric.mod.json`/`neoforge.mods.toml`/`*.gradle`, keep this repo's BoxLib-free versions and take upstream's other changes; if upstream bumps `boxlib_version`, diff `~/Desktop/Software/BoxLib` from `4ee4945` to the new commit and re-copy changed files, then update the recorded commit.
- [ ] **Step 4: Commit** `git commit -am "Document BoxLib merge and upstream sync procedure"`; push only after the user confirms.
