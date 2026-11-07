# Pixel art in the style of Duelyst, made with Claude Code

Research for hello1. Date: 2026-11-06. Local paths are relative to `/Users/quang.tran/works/self/game-projects/duelyst/` unless they are absolute. Measurement scripts are in the same scratchpad folder as this file: `plist_stats.py`, `sprite_stats.py`, `palette_union.py` and `crop_frames.py`. They use only the Python standard library.

## Summary

- **Sprite size.** Duelyst units are small pixel-art sprites on square canvases, usually 80, 100, 120 or 130 px. The visible character is about 44–71 px tall. The engine draws every sprite at **2x scale** (`CONFIG.SCALE = 2.0`) with **nearest-neighbour** filtering.
- **Animations.** Each unit has 6 animations. Median frame counts: breathing 14, idle 14, run 8, attack 23, hit 3, death 16. Nearly all animations play at **80 ms per frame (12.5 fps)**. Breathing is slower, 90–140 ms per frame.
- **Palette.** Each sprite uses **7–24 colours** and alpha is either fully on or fully off. 40% of the unit sheets are saved as indexed PNG, and 117 of them use only 16 colours or fewer. Common colours are shared across sprites: near-black blue-greys, white `#fcfcfc` and accent cyan/yellow. There is no strict master palette.
- **Outlines.** Sprites have no uniform black outline. Edges are drawn with the darkest colour of whatever part they border ("selective outline"). Light comes from the top, and shadows shift toward purple or blue.
- **Effects are done at runtime, not drawn into the sprites.** Ground shadows, glows, dissolves, bloom and lighting are GLSL shaders (72 files in `app/shaders`), and there are also 63 particle definitions. Battle-map backgrounds are **painted art**, not pixel art: one background uses 66,572 colours and has smooth gradients.
- **Licence.** The repo and its assets are released as **CC0 1.0**. We could legally reuse the art, but you've decided not to copy it, and CC0 does not waive trademark rights, so we stay away from the Duelyst names and logo anyway.
- **Claude can't make images directly.** Anthropic's docs say it "cannot generate, produce, edit… images". It can write code that writes pixels, and it can look at the result, though its vision is weak on images under 200 px. Scripts must upscale previews before Claude looks at them.
- **Recommended pipeline, free:** a palette file plus an ASCII grid per frame, then a small Python script (stdlib, or Pillow) that writes indexed PNGs, a spritesheet with JSON and 4x previews. Claude reviews the previews. The engine draws them with nearest filtering at an integer scale. Aseprite scripting is an optional upgrade, not a requirement. Aseprite costs $19.99, or you can compile it from source for personal use.
- **Paid options:** PixelLab is the only image-generation API I found whose official docs confirm pixel-art output, including animation. It costs about $0.008–$0.15 per call, and you own the output. It conflicts with the no-paid-tools decision, so it's listed only as an option.
- **MCP servers, Claude skills and plugins** are out of scope for this file. See pixel-art-skills.md.

---

## 1. Duelyst style spec, measured from the game files

### 1.1 Where the art lives

| Asset | Location | Format |
|---|---|---|
| Units and generals (animated) | `app/resources/units/` — 696 PNG + 696 plist | Cocos2d plist atlases. Each frame is named like `<unit>_<anim>_<NNN>.png` (`app/resources/units/f1_3rdgeneral.plist`) |
| Spell and artifact icons (animated) | `app/resources/icons/` — 746 files | Plist atlases. Frames are 48x48 (8,032 frames) or 34x34 (619 frames) |
| Spell and ability FX | `app/resources/fx/` — 423 files | Plist atlases, mostly RGBA. Frame delay 0.04–0.08 s (`app/data/resources.js:487-668`) |
| Particles | `app/resources/particles/` — 63 plist | Cocos particle-emitter definitions with additive blending (`app/resources/particles/ash.plist`) |
| Battle maps | `app/resources/maps/` | Painted layers at 1x and @2x: background 1280x720, middleground 1920x1080, foreground pieces |
| Board tile | `app/resources/tiles/tile_board.png` | 128x128, a single colour with soft alpha |
| Resource registry | `app/data/resources.js` | Maps each animation to its `img`, `plist`, `framePrefix` and `frameDelay` (format documented at `app/data/resources.js:29`) |

### 1.2 Canvas and character size

Source: `plist_stats.py` run over every unit plist, plus `sprite_stats.py` run on six units.

- **Frame canvas, counted over all unit frames:** 100x100 (22,010 frames), 80x80 (13,089), 130x130 (9,764), 120x120 (4,800), 110x110 (2,255), 140x140 (1,248). Canvases are always square. Small units use 80, typical units 100, generals and large units 120–140.
- **Character bounding box, measured on the first idle frame of each:**

  | Unit (file) | Canvas | Character height | Max width | Unique colours |
  |---|---|---|---|---|
  | `f1_general` | 100 | 44 px | 48 | 22 |
  | `f2_general` | 80 | 55–58 px | 45 | 16 |
  | `f4_general` | 80 | 47–51 px | 65 | 24 |
  | `f1_3rdgeneral` | 130 | 53–56 px | 57 | 24 |
  | `neutral_bastion` | 100 | 66–71 px | 35 | 15 |
  | `neutral_golemice` | 80 | 42–46 px | 39 | 8 |

- **Feet position.** The feet sit about 17–20 px above the bottom of the canvas. This matches `CONFIG.DEPTH_OFFSET = 19.5`, described in the code as "offset from bottom of sprites for depth calculations" (`app/common/config.js:572-573`).
- **Facing.** Units face right. The engine mirrors them with `setFlippedX` when they move left (`app/view/nodes/cards/EntityNode.js:1253`).
- **Sheet sizes.** Sheets are 1024x1024 (292), 2048x1024 (180) or 512x1024 (178). None are rotated (`plist_stats.py`).

### 1.3 Render scale and board

- `CONFIG.SCALE = 2.0`, commented "scale of sprites in view" (`app/common/config.js:570-571`). Card and unit nodes apply it (`app/view/nodes/cards/CardNode.js:538`).
- Unit sprites use `antiAlias: false` (`app/view/nodes/cards/EntitySprite.js:14`). This makes `BaseSprite._updateAliasing` call `setAliasTexParameters()`, which turns on nearest filtering (`app/view/nodes/BaseSprite.js:193-200`).
- Units have **no @2x files**. Maps and UI do have them (`CONFIG.RESOURCE_SCALES = [2]`, `app/common/config.js:32`). So unit pixel art is authored once at 1x and only ever scaled up.
- **Reference window and board:** the reference window is 1280x720 (`app/common/config.js:70`). The board is 9 columns x 5 rows (`app/common/config.js:72-73`). `CONFIG.TILESIZE = 95` screen px (`app/common/config.js:287`), which is about 47 art pixels at 2x. So a typical unit, 44–58 px tall, is about one tile high.
- **Window scaling.** The global window scale snaps to multiples of 0.25 (`app/common/config.js:26`, `:1289-1310`). There is also a "pixel perfect" resolution option (`app/common/config.js:36-41`). Aliasing is turned on only when the scale is "safe", meaning a multiple of 0.5 (`app/view/extensions/RenderingInjections.js:96-101`).
- **Board tilt.** The board is tilted with `CONFIG.XYZ_ROTATION = { x: 16.0 }` to give a sense of depth (`app/common/config.js:574-575`).

### 1.4 Animation conventions

Frame counts come from `plist_stats.py` over all 696 unit plists. Frame delays come from tallying `frameDelay` in `app/data/resources.js`.

| Animation | Units that have it | Median frames | Range | Frame delay |
|---|---|---|---|---|
| breathing (default resting loop) | 690 | 14 | 7–34 | 0.09–0.14 s. Most common: 0.12 s (230 units), 0.09 s (177) |
| idle (loop while the unit is selected) | 695 | 14 | 6–48 | 0.08 s (694 units) |
| run | 668 | 8 | 4–16 | 0.08 s (640 units) |
| attack | 693 | 23 | 5–54 | 0.08 s (688 units) |
| hit | 682 | 3 | 3–11 | 0.08 s |
| death | 691 | 16 | 5–53 | 0.08 s |
| caststart / castloop / castend (generals and casters only) | about 40 | 4 / 6 / 4 | — | — |

- **Which loop plays when.** Breathing plays when the unit is not selected, and idle plays when it is (`app/view/nodes/cards/UnitNode.js:39`, `:70-77`).
- **Spell icons** have an "active" animation with a median of 17 frames. Artifacts use 12 frames (`plist_stats.py` over `app/resources/icons`).

### 1.5 Palette, shading and outline

Measured with `sprite_stats.py` and `palette_union.py`. I also looked at frames enlarged 4x (`f2_general.png` and `golemice.png` in the scratchpad).

- **Colour counts.** Each unit uses 7–24 unique RGB colours, and the count is the same across all frames of an animation. Across the indexed sheets, the median palette has 17 entries (minimum 5, maximum 84). **Alpha is binary on every unit sampled:** pixels are fully opaque or fully transparent, with no anti-aliased edges.
- **PNG formats of the unit sheets:** 117 are 4-bit indexed (16 colours or fewer), 160 are 8-bit indexed and 419 are RGBA.
- **Shared colours.** I found 1,090 distinct colours across the 277 indexed sheets, so there is no locked master palette. Some colours recur in 25% or more of the sheets:
  - `#fcfcfc` (white highlight), in 195 sheets
  - `#14161f`, `#252a32` and `#0a0b0f` (dark blue-grey, used for the darkest outline and shadow tones)
  - `#c3cbdb` (cool light grey) and `#585651` (warm mid grey)
  - `#fce945` (yellow accent)
  - `#00fff0` and `#00b9ff` (cyan glow accents)

  `#4c6971` appears in every indexed sheet and is probably the transparent fill colour (unverified).
- **Outline.** There is no uniform black outline. On dark-armoured units (`f2_general`, `f4_general`), 38–46% of silhouette edge pixels are near-black (luminance under 50), using the shared blue-blacks `#0a0b0f` / `#14161f` or a deep navy `#00003c`. On light units (`f1_general`, `neutral_golemice`, `neutral_bastion`), 0–4% of edge pixels are dark, and the edge is the local mid or dark tone of each material. This is the selective-outline ("sel-out") approach.
- **Light direction and hue shift.** The light is at the top, slightly to the left: highlights sit on top and upper-left surfaces. This is my visual judgement from the 4x crops, not a value stored anywhere in the files. Shadows shift in hue rather than just getting darker:
  - Ice golem: cyan `#70d2dc` → light `#d3fff5`, shadow purple `#764b77`.
  - Lyonar general: skin `#d19c60` → shadow `#966264`; gold `#fce945` → shadow `#d19c60`.
  - Saturated accents (`#ff9000`, `#fffc00`, `#00b9ff`) are used for glowing eyes, gems and fire.
- **Ramps are short.** Each material uses 2–4 shades.

### 1.6 Shadows, glows and FX (runtime, not in the sprite)

- **Ground shadow:** a separate sprite, `resources/ui/unit_shadow.png`, drawn under each unit at opacity 200 (`app/view/nodes/cards/UnitNode.js:45-58`, `app/data/resources.js:5021`).
- **Cast shadows from dynamic lights:** done with shaders (`app/shaders/ShadowHighQualityFragment.glsl`, `ShadowLowQualityFragment.glsl`, `LightingFragment.glsl`). Quality is set by `CONFIG.SHADOW_QUALITY_*` (`app/common/config.js:1216-1218`).
- **Glow and highlight:** `GlowSprite` provides a shader glow with thickness, a colour ramp, pulse and noise (`app/view/nodes/GlowSprite.js:25-60`). Units inherit it through `EntitySprite` (`app/view/nodes/cards/EntitySprite.js:12`).
- **Other shader effects:** dissolve on death, bloom, distortion, levels, tint and more (`app/shaders/`, 72 files; options listed at `app/view/nodes/BaseSprite.js:180-190`).

**What this means for us:** the pixel sprite carries no glow, no shadow and no soft edges. All of that is added by the engine.

### 1.7 Style spec for hello1 (our parameters, inspired by the above)

| Parameter | Value |
|---|---|
| Unit canvas | 80x80 for small units, 100x100 standard, 128x128 for the General and bosses. Square, transparent background |
| Character height | 40–60 px. Feet about 18 px above the canvas bottom, centred horizontally. Facing right |
| Render | Integer 2x, nearest-neighbour, no mipmaps. Snap positions to whole pixels |
| Alpha | Fully on or fully off. No anti-aliased edges |
| Colours per sprite | 8–24, built from a shared project master palette of 32–48 colours |
| Ramps | 2–4 shades per material. Shift shadows toward blue or purple and highlights toward yellow or cyan |
| Light | Top-left, consistent across all sprites |
| Outline | Selective. Darkest local tone on edges. Near-black blue `#0f1118`-ish only for dark or metal parts (our own value, not Duelyst's) |
| Accents | 1–3 saturated glow colours (eyes, cores, energy) |
| Animations | breathing 8–14 frames at 120 ms; idle 8–14 at 80 ms; run 6–8 at 80 ms; attack 10–20 at 80 ms; hit 3 at 80 ms; death 10–16 at 80 ms. Smaller counts than Duelyst for a solo project |
| Icons | 48x48 |
| Backgrounds | Can be painted or high-colour. Doesn't need to be pixel art (Duelyst's isn't) |
| FX | Engine-side: shadow blob sprite, shader or tween glow, particles. Not baked into frames |

---

## 2. Pipelines with Claude Code

### 2.1 What Claude can and can't do

- **Can't output images.** Anthropic's docs: "Claude is an image understanding model only… it cannot generate, produce, edit, manipulate, or create images" (https://platform.claude.com/docs/en/build-with-claude/vision, FAQ).
- **Can look at images,** with stated limits. It "might hallucinate or make mistakes when interpreting… very small images under 200 pixels", and its counting and spatial outputs are approximate (same page, "Limitations"). So every preview must be **upscaled 4–8x with nearest-neighbour** before Claude looks at it, and checks such as colour count or alpha should be done by script, not by eye.
- **This report used exactly that loop.** Python scripts read the PNGs, wrote 4x crops, and Claude looked at the crops with the Read tool. It works with the standard library only; Pillow is not installed on this Mac.

### 2.2 Options

**A. Code-generated pixel grids (Python).** Claude writes a palette plus a character grid per frame, and a script turns them into PNGs.

- **Tools:** Pillow `Image.quantize(colors, palette=…, dither=Dither.NONE)`, `putpalette` and `resize(…, Resampling.NEAREST)` (https://pillow.readthedocs.io/en/stable/reference/Image.html), or the standard library alone (`zlib` + `struct`, as in `crop_frames.py`).
- **Pros:** free, reviewable in git as text, deterministic, and Claude edits grids precisely.
- **Cons:** Claude draws "blind" and has to correct itself from previews. It's slow for 20-frame attack animations. Expect simple, readable shapes rather than Duelyst-level detail.

**B. SVG with `shape-rendering="crispEdges"`.**

- **Pros:** easy to write.
- **Cons:** MDN says crispEdges is only a hint: the browser "*might* turn off anti-aliasing" (https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Attribute/shape-rendering). You still have to rasterise and re-quantise, so this is worse than A for sprites. Fine for quick mock-ups.

**C. Aseprite Lua scripts and the CLI.**

- **Scripting:** the API has `Image:drawPixel(x, y, color)`, `getPixel`, `Palette`, `Sprite`, `Layer`, `Frame`, `Cel`, `Tag`, `saveAs{filename, palette}`, and `resize{method='rotsprite'}` (https://www.aseprite.org/api/, https://www.aseprite.org/api/image).
- **CLI:** `aseprite -b --script x.lua --script-param k=v`, and `--sheet out.png --data out.json --format json-array --sheet-type packed`, plus `--scale`, `--color-mode indexed` and `--palette` (https://www.aseprite.org/docs/cli/).
- **Pros:** real layers, tags and onion-skin, and a human can open the file and touch it up. It also handles spritesheet export.
- **Cons:** Aseprite costs $19.99 minimum. The source can be compiled "for your personal purposes", and the compiled program may be used to make assets for commercial games. It can't be redistributed (https://www.aseprite.org/faq/). Not installed here.

**D. MCP servers, skills and plugins** (including Aseprite MCP integrations): out of scope here. See pixel-art-skills.md.

**E. Pixel-art image-generation APIs.** I list only services whose official docs confirm pixel-art output.

- **PixelLab** (https://www.pixellab.ai/pixellab-api):
  - Image generation up to 400x400 (Pixflux), 512x512 (Pixen, Pro).
  - Animation: animate-with-text, 256x256, up to 16 frames; skeleton animation 3–15 frames.
  - 8-direction rotation and inpainting.
  - Prices are listed as estimates, e.g. $0.0079 for a 64x64 Pixflux image, $0.03 for an 8-frame 256x256 animation, $0.08–0.15 for Pro calls.
  - For PixelLab's MCP integration, see pixel-art-skills.md.
  - **Terms:** "You own the copyrights to your creations, permitting usage for both commercial and non-commercial purposes". You may not use outputs to train other models. Use is subject to Open RAIL-M (https://pixellab.ai/termsofservice §1.2, §1.3, §3.3, §4.1).
  - **Pros:** the fastest route to many animated sprites.
  - **Cons:** paid. Output still needs palette clean-up. Getting a consistent house style across a roster is unverified.
- **Retro Diffusion:** the official site didn't return readable content (https://www.retrodiffusion.ai/), so its API, pricing and terms are **unverified**. Not recommended until checked.
- **General image models** (OpenAI, Gemini and others): I didn't find official docs promising true pixel-grid output, so they're **not listed**. If used, the post-processing below is mandatory.

### 2.3 Post-processing for clean pixel art (applies to every pipeline)

1. **Remove soft edges.** Threshold alpha to fully on or off (alpha ≥ 128 → 255, else 0). This matches Duelyst (§1.5).
2. **Lock the palette.** Map every pixel to the project palette with no dithering: Pillow `quantize(palette=pal_img, dither=Image.Dither.NONE)`. Then assert with a script that the colour count is at most 24.
3. **Fix AI output resolution.** For images from a generator, downscale to the native pixel grid with `Resampling.NEAREST`, and never bilinear. Pillow forces NEAREST for "P"-mode images (Pillow docs, `Image.resize`).
4. **Pack the spritesheet** with fixed-size cells: a ~30-line Python grid packer plus JSON. Free alternatives:
   - Aseprite `--sheet … --data … --sheet-type packed` (CLI docs).
   - Free Texture Packer: MIT, exports json, pixi, phaser, godot and cocos2d. It's in maintenance mode: "Only critical bugs will be fixed" (https://github.com/odrick/free-tex-packer).
5. **Engine rendering:** nearest filtering, integer scale, positions snapped to whole pixels.
   - **Godot:** `texture_filter = TEXTURE_FILTER_NEAREST` (https://docs.godotengine.org/en/stable/classes/class_canvasitem.html). Stretch mode `viewport` (or `canvas_items`), aspect `keep`, scale mode `integer` (https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html). The project-wide default setting name `rendering/textures/canvas_textures/default_texture_filter` is **unverified**: that part of the page was not read.
   - **Phaser:** `pixelArt: true` "Sets antialias to false and roundPixels to true" (https://docs.phaser.io/api-documentation/typedef/types-core).
   - **HTML/CSS:** `image-rendering: pixelated` (https://developer.mozilla.org/en-US/docs/Web/CSS/image-rendering).

---

## 3. Recommended pipeline (smallest viable, free)

**A: text sprite sources, a Python build script, and Claude reviewing upscaled previews.**

```
art/
  palette.hex              # 32–48 project colours, one hex per line
  units/<name>/<anim>/NN.txt   # one char per pixel, '.' = transparent, chars map to palette indices
tools/build_sprites.py     # txt -> indexed PNG frames, sheet.png + sheet.json, preview_4x.png; asserts rules
```

The loop, per unit:

1. Claude drafts the breathing frame 0 grid on a 100x100 canvas, following the spec in §1.7.
2. `build_sprites.py` writes the indexed PNG and a 4x preview, and asserts:
   - colour count ≤ 24
   - only palette colours used
   - alpha is binary
   - the feet line is at y ≈ 82
3. Claude reads the 4x preview and fixes the silhouette and shading. You give the final yes or no.
4. Make the other frames by copying and shifting rows: breathing is a 1–2 px bob, hit is a 3-frame white flash plus knockback. Start with **breathing, attack, hit and death only**, using small frame counts (8 / 10 / 3 / 10).
5. The engine adds the shadow blob, glow and particles at runtime, as Duelyst does.

**Why this one:**

- It costs nothing and needs no installs: the standard library is enough, and Pillow is optional.
- The sprite sources diff cleanly in git.
- Claude edits text far more reliably than it drives a GUI.
- It matches the Duelyst style, which is a small palette, binary alpha and effects in the engine.

**Upgrade path, only if needed:**

- Buy or compile Aseprite for hand touch-ups and onion-skinning.
- Script Aseprite with Lua through the CLI (`aseprite -b --script`) if you want Claude to work inside .aseprite files. For MCP-based options, see pixel-art-skills.md.
- Use PixelLab only if the no-paid-tools decision changes.

**Main risk:** Claude drawing grids by hand gives simple, readable sprites, but probably not Duelyst-level anatomy or complex 20-frame attacks. Keep the robots, the human and the salvaged parts chunky and geometric: robots suit that well. Make a test sprite before committing to the full art direction.

---

## 4. Licensing notes

- **Licence.** `LICENSE` and `COPYING` in the Duelyst repo are both the full CC0 1.0 Universal text (`LICENSE`, `COPYING`; `README.md:51-54`, "OpenDuelyst is licensed under the Creative Commons Zero v1.0 Universal license"). Press coverage of Counterplay's January 2023 release says the art assets were included under CC0 (https://gigazine.net/gsc_news/en/20230112-open-duelyst; https://gamingonlinux.com/2023/01/counterplay-games-makes-tactical-card-battler-duelyst-completely-open-source). These are secondary sources. The primary source is the repo licence itself, and there is no separate art licence in the repo.
- **Trademarks are not covered.** CC0 §4(a): "No trademark or patent rights held by Affirmer are waived" (`LICENSE:104-105`). Do not use the name "Duelyst", its logo (`app/resources/ui/brand_duelyst.png`), faction names or character names.
- **Our policy, by your choice:** we don't copy any Duelyst sprites, characters, palettes taken pixel-for-pixel, or names. We use only the measured parameters above (sizes, frame counts, timing, palette size, outline approach). Style parameters are not copyrightable expression in any case. That's a general principle, not legal advice; unverified for any specific jurisdiction.
- **Tools:**
  - Aseprite's EULA allows commercial use of assets you make with it, but not redistribution of the program.
  - Free Texture Packer is MIT.
  - PixelLab outputs are owned by you, with a no-training clause and Open RAIL-M terms.
  - Pillow's licence (MIT-CMU) is from memory and **unverified** here.

## 5. Sources

**Local (Duelyst repo, `/Users/quang.tran/works/self/game-projects/duelyst/`)**

- `LICENSE`, `COPYING`, `README.md`
- `app/common/config.js` (lines 26, 32, 36-41, 70-73, 287, 570-575, 1216-1218, 1256-1310)
- `app/data/resources.js` (lines 29, 95, 487-668, 5021; `frameDelay` tally over the whole file)
- `app/view/nodes/BaseSprite.js` (35, 180-200), `app/view/nodes/GlowSprite.js` (25-60), `app/view/nodes/cards/EntitySprite.js` (12-20), `app/view/nodes/cards/UnitNode.js` (30-90), `app/view/nodes/cards/EntityNode.js` (264-273, 1253), `app/view/nodes/cards/CardNode.js` (538), `app/view/extensions/RenderingInjections.js` (78-101)
- `app/shaders/` (72 files, including `ShadowHighQualityFragment.glsl` and `LightingFragment.glsl`)
- `app/resources/units/*.png|plist`, `app/resources/icons/`, `app/resources/fx/`, `app/resources/particles/ash.plist`, `app/resources/maps/battlemap0_*.png`, `app/resources/tiles/tile_board.png`

**Scratch scripts and outputs** (folder `/private/tmp/claude-502/-Users-quang-tran-works-self-game-projects-hello1/b53b5ec5-cc89-4cd3-8613-6fd5651281a4/scratchpad/`)

- Scripts: `plist_stats.py`, `sprite_stats.py`, `palette_union.py`, `crop_frames.py`
- Preview images: `f2_general.png`, `golemice.png`, `bg_crop.png`

**Web**

- Claude vision limits: https://platform.claude.com/docs/en/build-with-claude/vision
- Aseprite: https://www.aseprite.org/api/, https://www.aseprite.org/api/image, https://www.aseprite.org/docs/cli/, https://www.aseprite.org/faq/
- PixelLab: https://www.pixellab.ai/pixellab-api, https://pixellab.ai/termsofservice
- Retro Diffusion (unverified, page unreadable): https://www.retrodiffusion.ai/
- Pillow: https://pillow.readthedocs.io/en/stable/reference/Image.html
- Free Texture Packer: https://github.com/odrick/free-tex-packer
- Godot: https://docs.godotengine.org/en/stable/classes/class_canvasitem.html, https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html
- Phaser: https://docs.phaser.io/api-documentation/typedef/types-core
- MDN: https://developer.mozilla.org/en-US/docs/Web/CSS/image-rendering, https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Attribute/shape-rendering
- Secondary, for the CC0 announcement only: https://gigazine.net/gsc_news/en/20230112-open-duelyst, https://gamingonlinux.com/2023/01/counterplay-games-makes-tactical-card-battler-duelyst-completely-open-source

**Not found:** I found no artist interview or dev post from the original Duelyst team (Counterplay) on how the sprites were made. Any claim about their production tools is **unverified** and has been left out.
