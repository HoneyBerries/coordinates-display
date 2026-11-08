<!--
MODRINTH SUMMARY (paste into the separate "Summary" field, no formatting):
An actively updated fork of Coordinates Display that shows your coordinates, direction, biome and more on the in-game HUD.

PROJECT TITLE: Coordinates Display++
-->

[![Modrinth](https://raw.githubusercontent.com/Boxadactle/coordinates-display/3a9f7f5a3592c7888b387160595f89f74f756da2/img/modrinth.svg)](https://modrinth.com/mod/coordinates-display-plus-plus) [![Wiki](https://raw.githubusercontent.com/Boxadactle/coordinates-display/3a9f7f5a3592c7888b387160595f89f74f756da2/img/wiki.svg)](https://boxadactle.dev/wiki/coordinates-display/) [![GitHub](https://raw.githubusercontent.com/Boxadactle/coordinates-display/c47768fbc605863978b6c88ca3c4fcac4a5885da/img/github.svg)](https://github.com/HoneyBerries/coordinates-display)

# Coordinates Display++

**Coordinates Display++ is a community-maintained fork of [Coordinates Display](https://modrinth.com/mod/coordinates-display) by [Boxadactle](https://github.com/Boxadactle).** It keeps everything people liked about the original, a clean and configurable coordinates HUD, and aims to get new Minecraft versions and fixes out quickly.

![default hud](https://boxadactle.dev/img/coordinates-display/coordinates_hud.png)

## A thank-you to Boxadactle

Coordinates Display was created by **Boxadactle**, who designed the HUD, the configuration screens, position marking and death tracking, and maintained it for years. Hundreds of thousands of players use it because of that work. This fork exists only because of it, and all credit for the original mod belongs to Boxadactle. Please consider supporting the [original project](https://modrinth.com/mod/coordinates-display) and [its source](https://github.com/Boxadactle/coordinates-display) too.

This fork is **not affiliated with or endorsed by Boxadactle**. It is licensed under the same GPL-3.0 license as the original.

## Why this fork?

At the time of writing, the original mod's latest release targets Minecraft 26.2. This fork exists to keep Coordinates Display available on the newest versions, with frequent updates and bug fixes.

What's different from the original:

- **Updated to Minecraft 26.3.** I'll keep porting to new versions as they release.
- **BoxLib is bundled.** You no longer need to install the separate BoxLib library mod.
- **Fixes and new features**, like the day counter and time display, plus fixes to the time renderer and padding.
- **Fabric and NeoForge builds** published for every release.

> **Important:** Because BoxLib is built in, this fork is **not compatible with the standalone BoxLib mod** or mods that require it (for example BoxHUD). Remove them before installing. The BoxHUD integration will not activate while they are present.

## Features

Your HUD can show your precise coordinates, chunk coordinates, the direction you're facing, your current biome, the time of day and the Minecraft version.

### Multiple display modes
![min mode](https://boxadactle.dev/img/coordinates-display/minimum_hud.png) ![compass mode](https://boxadactle.dev/img/coordinates-display/compass_hud.png) ![hotbar mode](https://boxadactle.dev/img/coordinates-display/hotbar_hud.png)

Choose from several layouts: minimal, compass, hotbar and more.

### Highly configurable
![config screen](https://boxadactle.dev/img/coordinates-display/configuration/gui.png)

An in-game configuration screen lets you tweak everything without editing files.

### Easy positioning
![hud position screen](https://boxadactle.dev/img/coordinates-display/configuration/visual/change_position.gif)

Drag and scale the HUD to wherever you want it on screen.

### Custom colors
![colorized](https://boxadactle.dev/img/coordinates-display/configuration/custom5.png)

Change the color of the HUD text to match your style.

### Death position saving
![death position on death screen](https://boxadactle.dev/img/coordinates-display/configuration/death_pos/deathscreen.png)

Your death location appears on the death screen and is logged in chat when you respawn, so you can get your items back.

### Keybindings
![keybinds](https://boxadactle.dev/img/coordinates-display/keybinds/keybinds.png)

Customizable keybinds for the common actions.

### Position marking and sharing
![mark gui](https://boxadactle.dev/img/coordinates-display/markgui.png)

Mark and save locations, then share them with other players.

## Installation

1. Install [Fabric](https://fabricmc.net/) or [NeoForge](https://neoforged.net/) for your Minecraft version.
2. Download the latest Coordinates Display++ for your loader and version.
3. Drop the jar into your `mods` folder.

**Fabric users** also need [Fabric API](https://modrinth.com/mod/fabric-api). [Mod Menu](https://modrinth.com/mod/modmenu) is recommended for opening the config screen.

## Support and contributing

- Found a bug or want a feature? [Open an issue](https://github.com/HoneyBerries/coordinates-display/issues/new/choose).
- Pull requests are welcome, including [translations](https://github.com/HoneyBerries/coordinates-display/tree/main/common/src/main/resources/assets/coordinatesdisplay/lang).
- Source code: [GitHub](https://github.com/HoneyBerries/coordinates-display)
