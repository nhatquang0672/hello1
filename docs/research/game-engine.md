# Game Engine / Framework

Research notes for hello1. Researched 2026-11-08. Claims are cited with a URL or a local path. Anything marked **(inference)** is my own reasoning and not sourced.

Which engine or framework should hello1 use? The game is a turn-based tactics puzzle on a small grid (`docs/game-design.md` §1). The art is isometric pixel art, web first, with Aseprite spritesheets and in-engine effects (`docs/research/pixel-art/README.md`). The target screen is a phone browser in landscape. `docs/game-design.md` §6 says only that "a TypeScript web game was suggested but not discussed". PC (Steam) and phone app stores may come later; native wrappers were not researched in depth.

Bundle sizes marked "measured" were taken on 2026-11-08 by downloading the npm build from jsDelivr and running `wc -c` and `gzip -9`.

## Decision (2026-11-08)

**hello1 uses Phaser 4 + TypeScript for the current stage:** web first, sized for a phone browser in landscape. The research below is the basis.

Why Phaser:

1. **It covers the art direction out of the box.** Built-in Aseprite import keeps per-frame durations, so Wesnoth-style timing survives. `pixelArt: true` gives nearest filtering. It also has isometric tilemaps, tweens, particles and Glow/Shadow filters (§2).
2. **Small web download.** 352 KB gzipped, against about 5 MB or more for a Godot web build (§1, §2).
3. **Everything is TypeScript code.** It's diffable, and Claude Code can read, write and test all of it, with no editor to drive (§2, §10).
4. **Mature and active.** MIT, 40k+ stars, v4.2.1 released 9 Jul 2026 (§2).
5. **First-party agent skills** ship in the Phaser repo (see below).
6. **PC and app stores stay possible later** through Capacitor, Electron or Tauri **(inference)**.

Costs we accept:
- Integer scaling is our own code, about 20 lines (§2).
- PC and app builds need wrappers instead of a built-in export.
- Phaser 4 is new (v4.0.0, Apr 2026), so most examples online are v3 **(inference)**.

Ground rule: keep the game rules (grid, pushes, telegraphs, combos) in plain TypeScript with no Phaser import, and test them in Node. That keeps tests fast and switching engines cheap **(inference)**.

> **Important: use the Phaser skills when building the game.** Phaser ships official agent skills: 28 `SKILL.md` files covering game setup, scenes, sprites, animations, tilemaps, tweens, particles, filters, input, cameras, scaling and v3→v4 migration (https://github.com/phaserjs/phaser/tree/master/skills). They are vendored into `.claude/skills/`, pinned to commit `02d8931` (MIT, see `.claude/skills/PHASER-LICENSE.md` and `PHASER-SOURCE.md`). Before writing Phaser code, load the matching skill, such as `tilemaps`, `animations`, `scale-and-responsive`, `tweens`, `particles` or `filters-and-postfx`. Check `v4-new-features` and `v3-to-v4-migration` so no v3 API slips in. To update them, copy `skills/` again from a newer Phaser commit and update `PHASER-SOURCE.md`.

## Summary

1. **Godot 4 is current and free, but its web build is heavy and C# can't export to web.** Latest stable is 4.7.2 (18 Aug 2026), 4.8 is in beta 2 (https://godotengine.org/download/archive/). The docs say "Projects written in C# using Godot 4 currently cannot be exported to the web" (https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_web.html). The .NET web PR is still an open draft (https://github.com/godotengine/godot/pull/106125). A web build's `.wasm` was "around 40 MB uncompressed, and 5 MB compressed with Brotli" in 4.3 (https://godotengine.org/article/progress-report-web-export-in-4-3/). I found no official 4.7 figure. One project measured stock 4.7.2 at 39.5 MB raw, 6.9 MB Brotli (https://github.com/jhoffman-dev/sanctuary-farm/issues/148, third-party).
2. **Godot fixed its worst web problem.** Since 4.3, single-threaded export is "the preferred and now default way", so no COOP/COEP headers are needed (exporting_for_web, above). The docs still warn "Safari has several issues with WebGL 2.0 support that other browsers don't have" (same page).
3. **Godot has the best built-in fit for the art.** It has isometric TileSets, Y-sort, an `integer` stretch scale mode that "provides a crisp pixel art appearance", and a nearest default texture filter setting (`doc/classes/ProjectSettings.xml` at 4.7.2, lines 1096–1100, 3454–3456). Aseprite import is a well-used MIT plugin (https://github.com/viniciusgerevini/godot-aseprite-wizard).
4. **Phaser 4 is the strongest TypeScript option.** v4.0.0 shipped 10 Apr 2026, latest is v4.2.1 (9 Jul 2026), MIT (https://github.com/phaserjs/phaser/releases). It has built-in Aseprite animation import that keeps per-frame durations, isometric tilemaps, tweens, particles, and Glow/Shadow filters (Phaser source, section 2). The full minified build is 1.38 MB, 352 KB gzipped (measured).
5. **Phaser has no integer-scale mode.** `pixelArt: true` sets nearest filtering and `roundPixels` (`src/core/Config.js` line 395). Its scale modes are NONE, FIT, ENVELOP, RESIZE, EXPAND and two aspect modes, none integer (`src/scale/const/SCALE_MODE_CONST.js`). Integer zoom is a few lines of your own code **(inference)**.
6. **PixiJS is a renderer, not an engine.** It calls itself "an advanced, open-source 2D rendering engine" (https://pixijs.com/8.x/guides/getting-started/intro). Latest is v8.22.0 (1 Oct 2026), MIT. It has no Aseprite tag parser, no tilemap and no tweens in core, so you'd write or add them (section 3).
7. **Excalibur.js is TypeScript-first and fits well on paper, but is small and pre-1.0.** v0.32.0 (23 Dec 2025), BSD-2-Clause, 2,363 stars (GitHub API). It has an `IsometricMap` with automatic depth sort and an official Aseprite plugin that reads native `.aseprite` files (section 4). The plugin repo has 5 stars.
8. **Defold is free, tiny and strong on native, but has no isometric tilemap.** Lua only, free under the Defold License (https://defold.com/license/). "for HTML5 builds the empty project is now 1.06 MB" (https://defold.com/2026/05/04/Defold-1-12-4/). The isometric tilemap request is still open (https://github.com/defold/defold/issues/10660). Steam support is an official MIT extension (https://github.com/defold/extension-steam).
9. **Unity is the wrong tool for this game.** It is free under $200,000 revenue and the Runtime Fee is cancelled (https://unity.com/products/pricing-updates). It is heavy for a small 2D web game and not code-first **(inference)**.
10. **For Claude Code, TypeScript stacks win on workflow.** Everything is plain text, tests run in Node, and there's no editor to drive **(inference)**. Godot scenes (`.tscn`) are "mostly human-readable and easy for version control" (https://docs.godotengine.org/en/stable/engine_details/file_formats/tscn.html). The best-known Godot MCP server is third-party (Coding-Solo/godot-mcp, 6,562 stars, last push Apr 2026). I found no first-party Godot MCP server.
11. **Recommendation (inference), since adopted:** Phaser 4 + TypeScript, with game rules in a plain TypeScript module that has no Phaser import. Second: Godot 4 with GDScript, if native export and an editor matter more than web size and text-only workflow. Third: PixiJS + own code, if Phaser feels too heavy. Details in section 10; the decision is at the top.

## 1. Godot 4

| | Finding | Source |
|---|---|---|
| License, cost | MIT, free | GitHub API (`godotengine/godot`, license MIT, 119,756 stars) |
| Stable | 4.7.2-stable, 18 Aug 2026. 4.8 beta 2, 29 Oct 2026 | https://godotengine.org/download/archive/ |
| Web renderer | "Godot 4 can only target WebGL 2.0 (using the Compatibility rendering method)" | exporting_for_web |
| Threads | Since 4.3, single-threaded export is the default. A threaded build "will not run" without COOP/COEP headers or a service-worker workaround | exporting_for_web |
| Size | 4.3 `.wasm` "around 40 MB uncompressed, and 5 MB compressed with Brotli" | 4.3 web progress report |
| Size cuts | Disabling 3D cuts the binary "by about 15%". Needs a custom export template | https://docs.godotengine.org/en/stable/engine_details/development/compiling/optimizing_for_size.html |
| iOS / Safari | "Safari has several issues with WebGL 2.0 support that other browsers don't have". "The single-threaded export works very well on macOS and iOS too" | exporting_for_web |
| C# on web | "Projects written in C# using Godot 4 currently cannot be exported to the web." PR #106125 "[.NET] Add web export support" is an open draft, milestone 4.x | exporting_for_web; https://github.com/godotengine/godot/pull/106125 |

The 4.3 report also says audio in single-threaded builds was garbled on low-end machines "such as laptops or phones". The fix, sample playback, was "pretty much a prototype" at the time (https://godotengine.org/article/progress-report-web-export-in-4-3/). I did not check its status in 4.7 (**unverified**).

**Pixel-perfect and screen size.** For pixel art the docs say: "Set the stretch mode to viewport" and "Set the stretch scale mode to integer". For mobile they suggest `canvas_items` and `expand` (with a 720×1280 portrait example) (https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html). The two pieces of advice conflict for hello1; you'd pick viewport + integer with a small landscape base size **(inference)**. Integer mode floors the scale so the screen is "an integer multiple of the base viewport size" (`ProjectSettings.xml` line 1099). `rendering/textures/canvas_textures/default_texture_filter` sets nearest; `snap_2d_transforms_to_pixel` is "Useful for low-resolution pixel art games" (lines 2847–2848, 3454–3456).

**Isometric.** TileSets can use "Isometric, Half-Offset Square or Hexagon" shapes. Tiles have a "Y Sort Origin" that works when Y Sort is on for the `TileMapLayer` (https://docs.godotengine.org/en/stable/tutorials/2d/using_tilesets.html).

**Aseprite.** Aseprite Wizard imports into AnimationPlayer, AnimatedSprite2D and SpriteFrames, splits by tag, and is MIT. It is "only required during development" (https://github.com/viniciusgerevini/godot-aseprite-wizard). I did not confirm per-frame duration import (**unverified**, likely **(inference)**).

**Effects.** Tweens, shaders and particles are built in. The particles page recommends GPUParticles2D; CPUParticles2D "may perform better on low-end systems" (https://docs.godotengine.org/en/stable/tutorials/2d/particle_systems_2d.html). Which one behaves better on mobile web is **unverified**.

**Native.** Android export needs the Android SDK and OpenJDK 17 (https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_android.html). iOS: "You must export for iOS from a computer running macOS with Xcode installed" (https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_ios.html). Desktop export is built in.

**Claude Code fit.**
- `.tscn` scenes are text and diff well (tscn docs, above). Hand-editing them is fragile because of resource IDs and UIDs **(inference)**.
- Headless: `--headless` is "Useful for servers and with `--script`"; `--export-release` exports from the command line (https://docs.godotengine.org/en/stable/tutorials/editor/command_line_tutorial.html).
- Tests: GUT 9.7.1 supports Godot 4.7, has a CLI, 2.8k stars (https://github.com/bitwes/Gut). gdUnit4 tests GDScript and C#, runs "outside the Godot editor for CI/CD pipelines", 1.3k stars (https://github.com/MikeSchulze/gdUnit4).
- MCP: Coding-Solo/godot-mcp (6,562 stars, MIT, last push 16 Apr 2026) can launch the editor, run the project, read errors and add nodes. Setup is `claude mcp add godot -- npx @coding-solo/godot-mcp` (https://github.com/Coding-Solo/godot-mcp). It doesn't claim to be official. Others: ee0pdt/Godot-MCP (621 stars, last push Mar 2025), satelliteoflove/godot-mcp (184 stars, Oct 2026) (GitHub API).
- GDScript has far less public code than TypeScript, so Claude will make more API mistakes in it **(inference)**.

## 2. Phaser 3 / 4

| | Finding | Source |
|---|---|---|
| License, cost | MIT, free. 40,659 stars | `package.json` at v4.2.1; GitHub API |
| Stable | v4.2.1, 9 Jul 2026. v4.0.0, 10 Apr 2026. Last 3.x: v3.90.0, 23 May 2025 | https://github.com/phaserjs/phaser/releases |
| Phaser 4 changes | New node-based renderer; "Filter system" replaces v3 FX and masks; migration guide | https://github.com/phaserjs/phaser/releases/tag/v4.0.0 |
| TypeScript | Types ship in the package (`"types": "./types/phaser.d.ts"`) | `package.json` at v4.2.1 |
| Size | `phaser.min.js` 1,375,976 bytes, 352,441 gzipped | measured |
| Templates | `npm create @phaserjs/game@latest` gives "official project templates" | https://docs.phaser.io/phaser/getting-started/installation |

Start new work on Phaser 4, not 3. v3 has had no release since May 2025 (releases page, above) **(inference)**.

**Pixel-perfect and screen size.** `pixelArt: true` makes textures nearest-neighbour and sets `roundPixels` to true (`src/core/Config.js` lines 395–404 at v4.2.1). Scale modes are NONE, WIDTH_CONTROLS_HEIGHT, HEIGHT_CONTROLS_WIDTH, FIT, ENVELOP, RESIZE, EXPAND (`src/scale/const/SCALE_MODE_CONST.js`). None is integer. The config has a `zoom` value and an `orientation` option (https://docs.phaser.io/phaser/concepts/scale-manager). For integer scaling, compute `floor(min(screenW/baseW, screenH/baseH))` on resize and apply it as zoom **(inference)**.

**Isometric.** Tilemaps have an `ISOMETRIC` orientation "@since 3.50.0" (`src/tilemaps/const/ORIENTATION_CONST.js`). For an 8×8 board, plain sprites sorted by `x + y` are simpler than a tilemap **(inference)**.

**Aseprite.** `this.load.aseprite(...)` then `this.anims.createFromAseprite(key)` builds one animation per Aseprite tag. The doc comment lists the export settings (Packed, Trim, padding ≥ 1, "Tags" checked, item filename `{frame}`) (`src/animations/AnimationManager.js` lines 330–384 at v4.2.1). It reads each frame's `duration` (line 454), so Wesnoth-style uneven timing survives.

**Effects.** Core has `tweens`, `gameobjects/particles`, `gameobjects/shader` and `gameobjects/lights`. Phaser 4 filters include Glow, Shadow, Blur, Pixelate and ColorMatrix (`src/` directory listing at v4.2.1). That covers the glow, shadow and particle effects the art notes want **(inference)**.

**Native.** Capacitor wraps a web app as "a cross-platform native runtime" for iOS and Android (https://capacitorjs.com/docs). Phaser's docs don't mention Capacitor or desktop wrappers. Steam would need Electron or Tauri **(inference)**.

**Claude Code fit.**
- Scenes are TypeScript code, so everything is diffable **(inference)**.
- A `HEADLESS` renderer exists, "meant for unit testing", but "still absolutely relies on the DOM" (`src/const.js` lines 73–83). Keeping rules in plain TS and testing them with Vitest avoids that **(inference)**.
- MCP: `phaserjs/editor-mcp-server` is first-party but drives the Phaser Editor app; 36 stars, last push Sep 2025 (GitHub API). Not needed for a code-only workflow **(inference)**.

## 3. PixiJS (+ own logic)

| | Finding | Source |
|---|---|---|
| License, cost | MIT, free. 48,647 stars | GitHub API |
| Stable | v8.22.0, 1 Oct 2026; releases about monthly | https://github.com/pixijs/pixijs/releases |
| What it is | "an advanced, open-source 2D rendering engine", "Built on WebGL and optionally WebGPU" | https://pixijs.com/8.x/guides/getting-started/intro |
| Size | `pixi.min.js` 840,663 bytes, 236,771 gzipped | measured |
| Particles | v8 `ParticleContainer` renders "hundreds of thousands or even millions of particles". No emitter is documented on that page | https://pixijs.com/8.x/guides/components/scene-objects/particle-container |

**Aseprite.** The Spritesheet loader reads TexturePacker-style JSON with an optional `animations` map of frame names. The comment names TexturePacker, Shoebox and Spritesheet.js, not Aseprite (`src/spritesheet/Spritesheet.ts` lines 54, 180–184 at v8.22.0). Aseprite's JSON frames load, but tags and durations are ignored. `AnimatedSprite` accepts `{ texture, time }` frames (`src/scene/sprite-animated/AnimatedSprite.ts` lines 47–49). A ~30-line adapter from Aseprite tags to those frames fixes it **(inference)**.

**Everything else.** No isometric tilemap, scale manager or tween system in core, as far as I saw (**unverified**: I checked the intro page and source tree, not every package). You'd write iso projection, depth sort, integer scaling and tweens yourself, or add libraries **(inference)**. Native path: Capacitor, as for Phaser.

## 4. Excalibur.js

| | Finding | Source |
|---|---|---|
| License, cost | BSD-2-Clause, free. 2,363 stars, active (push 8 Nov 2026) | GitHub API |
| Stable | v0.32.0, 23 Dec 2025. Still 0.x | https://github.com/excaliburjs/Excalibur/releases |
| Size | `excalibur.min.js` 573,827 bytes, 147,217 gzipped | measured |
| Display | `DisplayMode.Fixed` (default), `FitScreen`, `FillScreen`, `FitContainer`, `FitScreenAndZoom` and more | https://excaliburjs.com/docs/screens |
| Pixel art | `antialiasing: false` is "useful for pixel art style art work you would like sharp". `pixelArt: true` is for "pretty" anti-aliased pixel art with blending on. `snapToPixel` exists | `src/engine/Engine.ts` lines 144–187, 241–243 at v0.32.0 |
| Isometric | `IsometricMap` class. `IsometricEntitySystem` sets `z = maxZ * elevation + pos.y` for depth sort | `src/engine/TileMap/IsometricMap.ts`, `IsometricEntitySystem.ts` lines 17–29 |
| Aseprite | `@excaliburjs/plugin-aseprite` loads "json or the native `.aseprite` file format" and "Supports tagged animations!" 5 stars | https://github.com/excaliburjs/excalibur-aseprite |
| Effects | Core has `Actions` (move/fade sequences) and `Particles` folders | `src/engine/` listing at v0.32.0 |

No integer-scale display mode is named on the screens page (**unverified** beyond that page). Excalibur is TypeScript from the ground up, which suits Claude Code. The small community means fewer examples and answers **(inference)**.

## 5. Defold

| | Finding | Source |
|---|---|---|
| License, cost | "The Defold License". "completely free to use", no royalties. You can't sell the engine itself | https://defold.com/license/ |
| Stable | v1.14.0, 12 Oct 2026 | https://github.com/defold/defold/releases |
| Language | "Write game logic using Lua" | https://defold.com/ |
| Platforms | HTML5, iOS, Android, Windows, macOS, Linux, Steam, consoles | https://defold.com/ |
| Web | Needs WebAssembly; WebGL 2 by default. The threaded build needs `SharedArrayBuffer`; ship `wasm-web` as fallback | https://defold.com/manuals/html5/ |
| Size | "for HTML5 builds the empty project is now 1.06 MB" | https://defold.com/2026/05/04/Defold-1-12-4/ |
| Project file | `game.project` "follows the INI file format standard" | https://defold.com/manuals/project-settings/ |
| Aseprite | Community "Asefold" exporter (Apache 2.0) | https://defold.com/assets/asefold |
| Steam | Official MIT extension | https://github.com/defold/extension-steam |

No built-in isometric tilemap: the tilemap manual mentions isometric only for the external Tiled editor (https://defold.com/manuals/tilemap/), and the feature request is still open (https://github.com/defold/defold/issues/10660). Lua is a fine language, but switching to it loses TypeScript's types and tooling **(inference)**.

## 6. Unity (brief)

- Runtime Fee cancelled 12 Sep 2024. Personal is free "up to $200,000 in revenue and funding"; splash screen optional with Unity 6. Pro is $2,310/yr per seat (https://unity.com/products/pricing-updates).
- Web supports "some mobile browsers": iOS Safari 15+, Chrome 58+ on Android (https://docs.unity3d.com/6000.2/Documentation/Manual/webgl-browsercompatibility.html).
- Not a fit: editor-centric, large web builds, and C# web builds are heavier than a JS bundle **(inference)**. Web build size **unverified**.

## 7. Plain TypeScript + Canvas

- Nearest scaling: set `imageSmoothingEnabled = false`. MDN: "useful for games and other apps that use pixel art" (https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/imageSmoothingEnabled).
- CSS `image-rendering: pixelated` scales by "nearest neighbor ... to the nearest integer multiple", then smooths the rest (https://developer.mozilla.org/en-US/docs/Web/CSS/image-rendering).
- Zero dependencies. You write the loop, input, animation, tweens, particles and glow yourself **(inference)**. For a turn-based game this is doable, but effects are where the time goes. Phaser gives them for free **(inference)**.

## 8. Others (brief)

- **LÖVE**: zlib license (`license.txt` in https://github.com/love2d/love). Latest release 11.5, Dec 2023 (GitHub API). I found no official web export (**unverified**). Weak fit for web first **(inference)**.
- **Bevy**: MIT/Apache-2.0 (GitHub API). README: "still in the early stages of development" and breaking releases "approximately once every 3 months" (https://github.com/bevyengine/bevy). Web via `wasm32-unknown-unknown` + `wasm-bindgen` (`examples/README.md`). Rust and churn make it a poor fit **(inference)**.
- **What the reference games used**: Duelyst's client is Cocos2d-JS 3.3 (`duelyst/docs/ARCHITECTURE.md` line 11). Wesnoth is C++ on SDL3 (`wesnoth/CMakeLists.txt` lines 550–552). Neither is a sensible choice for a new solo web game **(inference)**.

## 9. Comparison

| | Godot 4 (GDScript) | Phaser 4 | PixiJS 8 | Excalibur | Defold |
|---|---|---|---|---|---|
| License | MIT | MIT | MIT | BSD-2 | Defold License (free) |
| Latest stable | 4.7.2 (Aug 2026) | 4.2.1 (Jul 2026) | 8.22.0 (Oct 2026) | 0.32.0 (Dec 2025) | 1.14.0 (Oct 2026) |
| Web download (engine) | ~5 MB Brotli (4.3) | 352 KB gz (measured) | 237 KB gz (measured) | 147 KB gz (measured) | 1.06 MB empty project |
| Integer scaling | built in | own code | own code | not found | not checked |
| Nearest filter | setting | `pixelArt: true` | per texture | `antialiasing: false` | setting |
| Isometric | TileSet + Y-sort | iso tilemap | own code | `IsometricMap` | own code (request open) |
| Aseprite | plugin (MIT) | built in, keeps durations | adapter needed | official plugin | community exporter |
| Tweens/particles/shaders | all built in | all built in | particles + filters | actions + particles | built in |
| Native | built in | Capacitor / Electron | same | same | built in |
| Text + tests in Node | text scenes, GUT/gdUnit4 | yes | yes | yes | INI + Lua |

## 10. Recommendation (inference)

1. **Phaser 4 + TypeScript.** It covers every art need out of the box: Aseprite tags with per-frame timing, nearest filtering, tweens, particles, glow and shadow filters. Everything is code, so Claude Code can read, write and test all of it. Web download is small. Costs: write ~20 lines of integer scaling, and wrap with Capacitor for app stores.
2. **Godot 4 + GDScript.** Best tooling for isometric maps, pixel scaling and native export, and a visual editor. Costs: a ~5 MB compressed web download, Safari WebGL 2 caveats, an editor-centric workflow, and less reliable AI-written GDScript. Don't pick C# if web matters.
3. **PixiJS + own code.** Smallest real renderer, but you build animation timing, tweens, iso sorting and scaling. Only worth it if Phaser's API gets in the way.

Not shortlisted: Excalibur (good fit on paper, but pre-1.0 and a tiny ecosystem), Defold (Lua, no iso found), Unity, plain Canvas (too much effect work), LÖVE and Bevy.

**Deciding trade-offs.** Web-first and AI-friendly favours Phaser. Native app plus a visual editor favours Godot. Whichever you pick, keep the rules (grid, pushes, telegraphs, combos) in plain code with no engine import. Then tests are fast and switching engines later is cheap.

## 11. Unverified

- Godot single-threaded audio quality on phones in 4.7 (it was a prototype in 4.3).
- Godot web `.wasm` size for 4.7; the only official figure is from 4.3.
- Whether Aseprite Wizard keeps per-frame durations.
- GPUParticles2D vs CPUParticles2D behaviour on mobile web.
- Absence of a first-party Godot MCP server (I searched and found none).
- PixiJS: no tweens, tilemap or scale manager in core (checked intro and source tree only).
- Excalibur: no integer-scale display mode (checked the screens page only).
- Defold: nearest-filter setting values.
- Unity web build size.
- LÖVE official web export.
- Phaser Editor (needed for the first-party Phaser MCP) pricing.

## Sources

URLs and source paths are cited inline. Star counts, licenses and push dates come from the GitHub REST API (`api.github.com/repos/<owner>/<repo>`) on 2026-11-08. Source files were read from raw.githubusercontent.com at the tags named: Godot `4.7.2-stable`, Phaser `v4.2.1`, PixiJS `v8.22.0`, Excalibur `v0.32.0`. Local: `docs/game-design.md`, `docs/research/pixel-art/README.md`, `duelyst/docs/ARCHITECTURE.md`, `wesnoth/CMakeLists.txt`.
