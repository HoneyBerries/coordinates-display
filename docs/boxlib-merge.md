# BoxLib merge

BoxLib `common` module source and resources were copied into `:common`.

- BoxLib commit copied: `4ee4945` (from `~/Desktop/Software/BoxLib`)
- Date: 2026-11-07
- Package kept as `dev.boxadactle.boxlib`; `icon.png` not copied.
- BoxLib files are byte-identical copies, except:
  - `BoxLibNeoforge.java`: removed `@Mod`, `EventBusSubscriber` modid changed to `CoordinatesDisplay.MOD_ID`, removed `IConfigScreenFactory` registration, init moved into the `CoordinatesDisplayNeoforge` constructor.
  - Dev-only mixins unregistered (`TitleScreenMixin`, which adds the "Open Config" title-screen button).
  - `BoxLibModmenu` unregistered.

## Config location check (static)

- `ModConfig` is annotated `@BConfigFile("coordinatesdisplay")`; `BConfigHandler` resolves it to `<gameDir>/config/coordinatesdisplay.<filetype>` via `ClientUtils.getConfigFolder()`.
- `git diff c62e1fa..HEAD` does not touch `ModConfig.java` or any config-path logic.
- The vendored `BConfigHandler.java` and `ClientUtils.java` are byte-identical to BoxLib at `4ee4945`.
- Conclusion: the config file name and location are unchanged, so existing configs should load. This was checked by reading code only; no game was launched.

## Manual verification (to be done by a human)

Not yet performed. Repeat on both Fabric and NeoForge:

- [ ] Game launches; log shows no duplicate-class or mixin errors
- [ ] HUD renders
- [ ] Config screen opens (Mod Menu on Fabric, mods list on NeoForge)
- [ ] An existing `config/coordinatesdisplay.json` loads without resetting to defaults
- [ ] Keybinds work
- [ ] `/coordinates` runs
- [ ] Mark position works
- [ ] Death position shows
- [ ] No "Open Config" button on the title screen
- [ ] Installing the standalone `boxlib` mod alongside gives an incompatibility error

## Syncing with upstream

Upstream: `https://github.com/Boxadactle/coordinates-display` (remote `upstream`; `origin` is the fork). Use merge, not rebase, because `main` is pushed to `origin`.

```
git fetch upstream
git merge upstream/dev
```

- On conflicts in `gradle.properties`, `fabric.mod.json`, `neoforge.mods.toml` or `*.gradle`, keep this repo's BoxLib-free versions and take upstream's other changes.
- If upstream bumps `boxlib_version`: diff `~/Desktop/Software/BoxLib` from `4ee4945` to the new commit, re-copy the changed files into `:common` (preserving the edits listed above), and update the recorded commit at the top of this file.

Dry-run probe on 2026-11-07 (`git fetch upstream`; `git merge-tree --write-tree --name-only HEAD upstream/dev`; `git rev-list --left-right --count HEAD...upstream/dev`):

- `main` is 6 commits ahead of and 0 behind `upstream/dev` (`c62e1fa`).
- Merge-tree reported a clean merge with no conflicting files (trivially, since upstream has nothing new).
