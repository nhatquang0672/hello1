# hello1 pixel-art style guide

What Duelyst and Battle for Wesnoth do well, measured from their files, turned into rules Claude follows when drawing in Aseprite through the `aseprite-ai-artist` (pebbly) plugin. We learn the craft only: no sprites, characters, palettes copied pixel-for-pixel, or names from either game.

Date: 2026-11-06. Path shorthands used below:

- `DUELYST/` = `/Users/quang.tran/works/self/game-projects/duelyst/`
- `WESNOTH/` = `/Users/quang.tran/works/self/game-projects/wesnoth/`
- `PEBBLY/` = `/private/tmp/claude-502/-Users-quang-tran-works-self-game-projects-hello1/b53b5ec5-cc89-4cd3-8613-6fd5651281a4/scratchpad/repos/pebbly/` (plugin v0.8.2, `package.json:3`)
- `EARLIER/` = `/private/tmp/claude-502/-Users-quang-tran-works-self-game-projects-hello1/b53b5ec5-cc89-4cd3-8613-6fd5651281a4/scratchpad/pixel-art-research.md` (the earlier Duelyst report; its numbers are reused here, not re-measured)
- `SCRATCH/` = `/private/tmp/claude-502/-Users-quang-tran-works-self-game-projects-hello1--claude-worktrees-pixel-art-research/b53b5ec5-cc89-4cd3-8613-6fd5651281a4/scratchpad/` (new scripts in `scripts/`, outputs in `out/`, all run with `python3 -I`, stdlib only)

---

## 1. Summary

- **Both games keep the sprite clean and put the "juice" in the engine.** Duelyst adds glow, shadows, dissolve and hit sparks with shaders and FX sprites. Wesnoth slides the attacker toward the target, and flashes the defender red, in code. Sprites carry no glow and no soft edges.
- **Small canvases, one view, one facing.** Duelyst: square 80–140 px canvases, units face right, drawn at 1x and shown at 2x. Wesnoth: 72×72 on a 72 px hex for 233 of 263 base unit sprites, facing lower-right in 3/4 view.
- **Hard silhouettes.** Duelyst alpha is fully on or off. In Wesnoth the only partial alpha is a baked ground shadow: 236 of 263 base sprites have exactly one translucent colour, at alpha 153 (60%).
- **Dark, never-black edges.** Wesnoth's silhouette edge is 78% near-black (median), most often `#101010`; its wiki says pure `#000000` must never be used. Duelyst uses a selective outline: near-black only on dark parts, local dark tones elsewhere.
- **Short hue-shifted ramps.** Duelyst uses 7–24 colours per sprite with 2–4 shades per material. Wesnoth uses a median of 26 colours, with 4–8 shades per material and purple as the default shadow.
- **Two timing philosophies.** Duelyst draws many frames at a flat 80 ms: median attack is 23 frames, with impact at about 61% of it. Wesnoth draws few frames with hand-set durations: the median attack frame is 100 ms, about 5 frames, and the hit lands at time 0, typically about 300 ms after the attack starts.
- **The recolour mechanism.** Wesnoth recolours only 19 exact magenta values, preserving their brightness. Duelyst marks the owner with a glow or tint on the whole sprite instead.
- **Rules for hello1 (section 5):** 80×80 standard canvas, binary alpha, a 32-colour master palette loaded from a file, ≤24 colours per sprite, selective dark outline that is never black, one light direction, a reserved 4-shade team-key ramp, few frames with variable timing, and no baked FX or shadows.
- **pebbly has no project-rules file.** It loads its rulebook only from its own package (`PEBBLY/src/server.ts:50-52`, `PEBBLY/src/lib/skills.ts:69-70`). Load our rules through a Claude Code project rule (`.claude/rules/pixel-art.md`, always loaded), a palette file (`palette op=load`), and the brief pasted into each request (section 6).
- **Licensing.** Wesnoth art is GPL v2+, or CC BY-SA 4.0 for contributions from 30 July 2017 on. Learning the style binds us to nothing, but reusing a sprite would. Duelyst is CC0, but its trademarks are not waived.

---

## 2. What makes Duelyst's art work

The numbers come from `EARLIER/` §1 unless another source is given. This section adds the animation feel and the FX split.

### 2.1 Concepts already measured (from `EARLIER/` §1)

| Concept | Number | Source |
|---|---|---|
| Canvas | square: 100 (22,010 frames), 80 (13,089), 130, 120, 110, 140 | `EARLIER/` §1.2 |
| Character height | 44–71 px. Feet about 17–20 px above the canvas bottom (`CONFIG.DEPTH_OFFSET = 19.5`) | `EARLIER/` §1.2; `DUELYST/app/common/config.js:572-573` |
| Render | 1x art at `CONFIG.SCALE = 2.0`, nearest filtering, no @2x unit files | `DUELYST/app/common/config.js:570-571`; `DUELYST/app/view/nodes/cards/EntitySprite.js:14` |
| Facing | right, mirrored with `setFlippedX` | `DUELYST/app/view/nodes/cards/EntityNode.js:1253` |
| Colours | 7–24 per sprite. Median indexed palette is 17. No master palette (1,090 distinct colours across 277 sheets) | `EARLIER/` §1.5 |
| Alpha | binary on every sampled unit | `EARLIER/` §1.5 |
| Outline | selective: 38–46% near-black edge on dark units, 0–4% on light units | `EARLIER/` §1.5 |
| Light and hue shift | top or top-left (visual judgement only). Shadows go toward purple or blue. Ramps have 2–4 shades | `EARLIER/` §1.5 |
| Frames | breathing 14, idle 14, run 8, attack 23, hit 3, death 16 (medians). 80 ms, breathing 90–140 ms | `EARLIER/` §1.4 |

### 2.2 Animation feel, read from the frames (new)

`SCRATCH/scripts/attack_agg.py` takes 40 random units and, for each attack frame, finds the frame with the largest pixel change from the previous frame (the impact) and the frame with the furthest forward reach. Output: `SCRATCH/out/attack_agg.txt`. Per-frame detail for three units is in `SCRATCH/out/anim_feel.txt` (from `SCRATCH/scripts/anim_feel.py`).

- **Timing is flat, so the feel is drawn into the spacing.** Every attack frame is 80 ms (`DUELYST/app/data/resources.js:5136`, `:8045`; 688 of 693 attacks, `EARLIER/` §1.4). There are no per-frame durations.
- **Long wind-up, sudden strike, long recovery.**
  - Median attack: 22.5 frames. The impact frame is frame 13.5, at about 61% of the animation, with 8 frames (≈640 ms) after it.
  - Ice golem: frames 0–4 barely move (pixel change ≈500–830 per frame, reach x 53–57). Frame 5 jumps (change 1,750, reach x 79), then 4 recovery frames.
  - Bastion: 13 quiet frames, then the strike at frame 14 (change 3,256, reach 65 → 80) (`SCRATCH/out/anim_feel.txt`).
- **Holds are "moving holds", not repeated frames.** Only 3 of 40 attacks contain an exact duplicate frame. Instead, wind-up frames change by a few hundred pixels: f1_general frame 12 changes by only 89 pixels (`SCRATCH/out/attack_agg.txt`, `SCRATCH/out/anim_feel.txt`).
- **The strike frame is a smear.** On f1_general, 39% of the opaque pixels are near-white (min channel ≥ 240) on both strike frames (5 and 15), against 3–5% elsewhere (`SCRATCH/out/anim_feel.txt`).
- **The engine syncs damage to the drawn impact.** Each unit has `attackDelay`, the seconds from attack start to applying the hit to the target. `GameLayer` adds it to the action delay (`DUELYST/app/view/layers/game/GameLayer.js:3388-3392`). Common values are 1.2 s (172 cards), 0.6, 0.4, 0.5 and 0.3. For example, f1_general uses 0.5 s ≈ frame 6 at 80 ms, and its biggest change is at frames 5–6 (`DUELYST/app/sdk/cards/factory/misc/tutorial.coffee:73-75`; tally via `grep` over `DUELYST/app/sdk`). `attackReleaseDelay` times the swing sound (`DUELYST/app/view/nodes/cards/UnitNode.js:248-254`).
- **Hit is 3 frames (240 ms).** The impact itself is sold by FX on the target, not by the defender's frames (next section).

### 2.3 How the FX layer carries the juice (new)

- **Impact FX sit on the target.** These are stacked FX sprites. For example, a faction general's `UnitAttackedFX` is an orange impact sprite plus blue collision sparks offset by (-10, 10) (`DUELYST/app/data/fx.js:5000-5008`). The FX atlases run at 40–80 ms per frame (`EARLIER/` §1.1).
- **Spawn FX are layered.** A recall flash, a faction effect, ground smoke, and a dynamic `Light` with radius 270, intensity 9 and a warm colour (`DUELYST/app/data/fx.js:8-30`).
- **Light, glow and ownership are runtime effects.**
  - Shader-driven: 35 `FX*Sprite` files among 42 in `DUELYST/app/view/nodes/fx/` (rays, shockwave, distortion, dissolve, shadow blob, lens flare), 72 GLSL files (`EARLIER/` §1.6), and `GlowSprite` (`DUELYST/app/view/nodes/GlowSprite.js:25-60`).
  - Ownership is a pulsing highlight: blue `{70,90,255}` for the player, red `{255,0,0}` for the opponent (`DUELYST/app/common/config.js:751`, `:759`; `DUELYST/app/view/nodes/cards/EntityNode.js:1961-1971`). There is also an optional owner tint via `setColor` (`DUELYST/app/view/nodes/cards/EntityNode.js:555-560`).
- **Particles** are 63 Cocos emitter plists with additive blending (`EARLIER/` §1.1).
- **Consequence:** the sprite keeps binary alpha and a small palette, while light, glow, team identity and impact flashes stay engine-side and can be tuned without redrawing.

---

## 3. What makes Wesnoth's art work

Sprite stats come from `SCRATCH/scripts/wes_sprite_stats.py` over the 263 base images named by `image=` in `WESNOTH/data/core/units/**/*.cfg` (list: `SCRATCH/out/wes_unit_base2.txt`; output: `SCRATCH/out/wes_survey.txt`). Animation stats come from `SCRATCH/scripts/wes_anim_timing.py` (`SCRATCH/out/wes_anim_timing.txt`). That timing tally is a regex pass over WML and is approximate: it skips macro-generated frames.

### 3.1 Canvas, scale, view

- **Hex and canvas.** `tile_size = 72` (`WESNOTH/src/game_config.cpp:55`). 233 of 263 base sprites are 72×72; the others are 72×80, 90, 98 or 70 (`SCRATCH/out/wes_survey.txt`). The wiki: canvas 72×72, wider "try not to do this if at all possible"; "All but exceptionally large units should be contained in the hex" (https://wiki.wesnoth.org/Creating_Unit_Art).
- **Character size and placement.** Median opaque height is 49 px and median feet row is y = 61 (the wiki says "around 55 pixel down from the top edge"). Units are centred horizontally.
- **View and facing.** 3/4 view. "Unit must look in the lower right (… the artist's lower right …)". The engine mirrors the sprite for other directions (same wiki page). Directional extras such as `-n`, `-ne` and `-s` exist only for some attacks and stands (`WESNOTH/data/core/units/humans/Loyalist_Spearman.cfg`).
- **Light.** "Light comes from the artist's … upper right (for shading and highlights)" (same wiki page). The survey agrees: the luminance-weighted centroid sits above the shape centroid in 222 of 263 sprites, and slightly right (median +0.36 px).
- **Medium.** "only units drawn in pure pixel art will be accepted into mainline" (same wiki page). The 4x preview `SCRATCH/out/wes_4x.png` shows three representative sprites.

### 3.2 Palette, outline, edges

- **Colours.** Median 26 opaque colours per sprite. p10 = 13, p25 = 20, p75 = 32, p90 = 40 (`SCRATCH/out/wes_survey.txt`). This includes team-colour and shadow colours.
- **Shade guidance** (https://wiki.wesnoth.org/Choosing_your_palette):
  - "Any material … should have at least 4 shades"; bigger surfaces up to 6, the dominant one "around 8 … but try to keep the number low".
  - "The default shadow for Wesnoth is purple."
  - Complementary shadows make a surface "shine a little brighter". Green or blue reflections on metal.
  - Highlights "very sparely (single pixels)".
  - "advise against using the same shades in the same image on different surfaces".
- **Outline.** Median 78% of silhouette edge pixels have luminance < 50. The most common edge colour is `#101010` (61 sprites), then `#131313`, `#09060a` and `#12120d` (`SCRATCH/out/wes_survey.txt`).
  - The wiki: outlines "made with a darker, but similar color to what they're surrounding", and "pure black (RGB #000000) should _never_ be used to outline a sprite" (Creating_Unit_Art).
  - Choosing_your_palette: "Blackish outline (Don't use black; The outline can be the same colour for the whole unit)".
  - So in practice this is a near-black, mostly uniform outer outline, lighter where it meets light materials (71–93% dark on the sampled humans, `SCRATCH/out/wes_sample.txt`).
- **Alpha.** The silhouette is hard-edged. The only translucency is the baked ground shadow:
  - 236 of 263 sprites have exactly one partial-alpha colour, at alpha 153 (60%). It is `#170035`/`#160034`, sitting in a band under the feet (spearman: rows 53–57; opaque body ends at row 56). 16 sprites have none (`SCRATCH/out/partial_alpha_all.txt`, `SCRATCH/out/partial_alpha.txt`).
  - This matches the wiki's "preferred shadow color … 160034" and "All of the shadow pixels are given an opacity of 60%" (https://wiki.wesnoth.org/Creating_Shadows_Under_Units).
- **Anti-aliasing: the wiki and the shipped sprites disagree.** The wiki tutorial recommends edge pixels "the same color as the edge, but with variable transparency" (https://wiki.wesnoth.org/How_to_Anti-Alias_Sprite_Art). The measured mainline base sprites do not do this on the silhouette (above). Any AA is therefore internal and opaque (inferred from the alpha data; not separately measured).

### 3.3 Team colour (recolour-friendly palette)

- **Key colours.** The artist paints team-coloured parts with 19 exact magenta key values: `magenta=F49AC1,3F0016,…,FDE9F1` (`WESNOTH/data/core/team-colors.cfg:329-331`). `unit_rgb=magenta` makes this the default for units (`WESNOTH/data/game_config.cfg:45`). There are 48 `[color_range]` teams, each defined by mid, max (highlight), min (shadow) and minimap colours (`WESNOTH/data/core/team-colors.cfg:2-80`).
- **The algorithm** (`WESNOTH/src/color_range.cpp:32-69`):
  - The first key (`F49AC1`) is the reference.
  - Any key darker than it maps between the team's min and mid, by the ratio of average brightness.
  - Any lighter key maps between mid and max.
  - Only exact matches change (`generate_color_mapping`, `:91-102`). Wiki: "these shift ONLY the precise RGB values given", "use a tolerance of 0" (https://wiki.wesnoth.org/Team_Color_Shifting).
- **Applied with image path functions.** `~TC(side,palette)` (`WESNOTH/src/image_modifications.cpp:543-576`) and `~RC(magenta>color)` (e.g. `WESNOTH/src/units/drawer.cpp:53-62`).
- **Amount.** 225 of 263 sprites contain key pixels, at a median 7% of opaque pixels (4–14% on the sampled humans) (`SCRATCH/out/wes_survey.txt`, `SCRATCH/out/wes_sample.txt`). These are sashes, shields, trims and plumes (see `SCRATCH/out/wes_4x.png`): small accents, not the whole body.

### 3.4 Animation

All of this is in WML, with per-frame durations, e.g. `image="…fencer-stand-[1~8].png:[200,80*6,200]"` (`WESNOTH/data/core/units/humans/Loyalist_Fencer.cfg:28`).

| Animation | Median ms/frame | Common values | Frames per block (median) | Notes |
|---|---|---|---|---|
| standing (loop) | 200 | 200, 150, 300, 80, 250, 400 | 8 | first and last frames often held longer, e.g. `[200,80*6,200]` |
| idle (occasional flourish) | 110 | 100, 200, 150 | 11 | plays at a random interval of 20–40 s (`WESNOTH/src/units/animation_component.cpp:34-43`) |
| attack | 100 | 100, 50, 75, 150, 80, 200 | 5 | `start_time` median −300 ms (−200 to −550). **Time 0 is the moment of the hit** |
| defend | 83 | macro: base 1 ms → reaction 250 ms → base 1 ms | 2–3 | `start_time=-126` (`WESNOTH/data/core/macros/animation-utils.cfg:82-101`) |
| death | 100 | 100, 28, 150 | 7 | |

Source for the table: `SCRATCH/out/wes_anim_timing.txt`.

- **Feel is set by duration, not frame count.**
  - Spearman melee: `[100,75]` wind-up, a 50 ms swoosh frame composited with `~BLIT(…swoosh…)`, then `[50,150]` recovery, with `offset="0:180,0~0.5:145,0.5~0:200"` sliding the unit half a hex toward the target and back (`WESNOTH/data/core/units/humans/Loyalist_Spearman.cfg:116-136`).
  - Fencer lunge: 10 frames at 50 ms with an offset curve that first pulls back to −0.25 (anticipation), then lunges to 0.55 (`WESNOTH/data/core/units/humans/Loyalist_Fencer.cfg:70-80`).
- **The engine adds juice.**
  - Default melee attack: the unit slides `0~0.6:150,0.6~0:150` around the hit (`WESNOTH/src/units/animation.cpp:576-578`).
  - On a hit the defender gets a 225 ms red blend, `0.0,0.5:75,0.0:75,0.5:75,0.0` with colour `{255,0,0}` (`WESNOTH/src/units/animation.cpp:566-571`, `:796-799`).
- **Wiki guidance** (https://wiki.wesnoth.org/Creating_Unit_Art, https://wiki.wesnoth.org/Basic_Animation_Tutorial):
  - "about 6 frames is optimal" for attacks (minimum 4); defence "usually 2 frames long not including the base frame".
  - "always err on the side of over-exaggerating things like windups".
  - Strong strokes go "clean through" the target.
  - Moving only the "active element" looks robotic.
  - Redraw a part after "about a 30-45 degree turn".
  - Select-and-slide only "for the tiniest motions".
  - Hand-drawn motion blurs on fast weapons.

### 3.5 Portrait and sprite separation

- **Two separate assets.** Each unit has a battlefield sprite (`image=`, 72×72 PNG) and a separate painted portrait (`profile="portraits/humans/fencer.webp"`, `WESNOTH/data/core/units/humans/Loyalist_Fencer.cfg:5-6`).
- **Portraits** are 400×400 or 500×500 WebP (`SCRATCH` one-liner over `WESNOTH/data/core/images/portraits/humans/*.webp` VP8X headers), and not pixel art. The portrait carries identity and detail; the sprite only has to read on the board.
- **Wesnoth uses separate PNGs for each frame** ("Each frame of a unit animation … is stored as a separate .png file", https://wiki.wesnoth.org/Art_Tutorials).

### 3.6 In-repo docs

- I found no art guide in the repo (no README under `WESNOTH/data/core/images/units`, and `WESNOTH/doc/` holds only GUI and design docs). The tool `WESNOTH/data/tools/TeamColorizer` and the swatch `WESNOTH/images/tools/magenta_team_color_palette.png` exist; I didn't read either.
- Forum tutorials by core artists (e.g. "Zerovirus's Spriting Workflow", http://forums.wesnoth.org/download/file.php?id=56284) were **not read: unverified**.

---

## 4. Side-by-side

| | Duelyst | Wesnoth | hello1 (section 5) |
|---|---|---|---|
| Canvas | square 80 / 100 / 120–140, art at 1x, shown 2x | 72×72 (= hex), shown 1x (zoomable) | 80×80 standard, 64 small, 128 big; shown at integer 2x |
| Body height / feet | 44–71 px / feet 17–20 px above the bottom | median 49 px / feet ≈ y 55–61 | 40–56 px / feet on row 62 (18 px above the bottom) |
| View, facing | side-on with slight top, faces right | 3/4, faces lower-right | side-on with slight top, faces right |
| Palette | 7–24 colours, no master palette | median 26 (13–40), ramps of 4–8 | 32-colour master file, ≤24 per sprite, ramps of 3–4 |
| Outline | selective: dark on dark parts, local tone elsewhere | near-black (`#101010`-ish) outer, never `#000000` | selective, near-black blue-purple outer on dark parts, never `#000000` |
| Light | top / top-left (visual) | artist's upper right (official) | upper-left, fixed |
| Alpha | binary | binary + one 60% purple ground shadow | binary; shadow is engine-side |
| Animation | many frames, flat 80 ms; attack ≈ 23 frames, impact ≈ 61% | few frames, per-frame ms; attack ≈ 5 frames at 50–200 ms, hit at t=0 | few frames, per-frame ms; record the impact ms |
| Recolour | none in sprites; owner glow or tint at runtime | 19 exact magenta keys, brightness-mapped | 4 exact key colours + engine highlight |
| FX | shaders, FX sprites, particles, lights | engine offset slide, red hit blend, `~BLIT` swoosh overlays, halos | engine-side; one smear frame allowed in the sprite |

---

## 5. hello1 style rules

### How to read the rules

**Tags** say who checks each rule:

- **[V]** = pebbly `validate` reports it. Run `validate strict=true`.
- **[T]** = checkable with another pebbly tool: `sprite_info`, `palette op=analyze`/`get`, or `look op=ascii`/`filmstrip`.
- **[E]** = eye review (`look op=preview`, or the `pixel-critic` agent).

`validate` checks off-palette colours, strays, outline gaps, banding, partial alpha, empty layers, missing tags and uniform timing. It is blind to pillow shading, brightness-only ramps, light direction, doubled outlines and volume drift (`PEBBLY/rules/92-generated-art-tells.md:92`; implementation `PEBBLY/extension/ai-artist.lua:3692-3901`).

**Override** marks where a rule replaces a pebbly default.

### Canvas and placement

1. **Canvas.** Make every unit canvas square and transparent: 80×80 for standard units, 64×64 for small units and pickups, 128×128 for bosses. [T `sprite_info`]
   - **Override:** pebbly's brief default is 32×32 (`PEBBLY/skills/brief/SKILL.md`, table).
2. **Placement.** Draw the character 40–56 px tall on an 80 canvas, centred horizontally, with the lowest foot pixel on row 61 (0-based). Use the same foot row in every frame of every animation. [T `look op=ascii`]
3. **Facing and view.** Face right, side-on with a slight top-down tilt. Never draw left-facing frames; the engine mirrors. [E]
   - **Override:** pebbly's default view is 3/4 (`PEBBLY/skills/brief/SKILL.md`).
4. **Silhouette first.** Fill the whole shape with one colour and check it reads at 1x and 2x before shading. Head, weapon and the role-defining prop each need a distinct bump in the silhouette. pebbly already teaches this; follow `PEBBLY/rules/03-silhouette-and-form.md` and `PEBBLY/rules/14-readability-and-scale.md`. [E]

### Palette and colour

5. **Master palette.** Load the hello1 master palette file (32 colours) before drawing, with `palette op=load path=<absolute path>`. Never use `preset`.
   - **Override:** pebbly falls back to PICO-8 when nothing is said (`PEBBLY/skills/palette/SKILL.md:20`, `PEBBLY/skills/new/SKILL.md:34`).
   - Check: `validate` [V] flags off-palette pixels; `palette op=analyze` [T] flags near-duplicate entries (ΔE < 3).
6. **Colour count.** Use at most 24 distinct colours per sprite, counting all frames. [T `palette op=get`: count entries with pixels > 0]
7. **Ramps.** Use 3–4 shades per material ramp (5 only for the dominant material on 128 px sprites). Shift shadows toward blue or purple and highlights toward warm yellow. A ramp that only changes brightness fails.
   - [E] Validate can't see this. Follow `PEBBLY/rules/01-palette-and-color.md` and `PEBBLY/rules/20-color-for-pixel-art.md`; the `palette op=ramp` tool already hue-shifts (`PEBBLY/src/tools/palette.ts:65`).
8. **Saturation.** Keep saturated "glow" colours to 1–3 accents per sprite (eyes, cores, gems), each ≤ 3% of opaque pixels. [T `palette op=get`]
9. **Team keys.** Paint owner-coloured parts (sash, shield face, trim) only with the 4 team-key colours `#4A0D3F #8A1A76 #C42DA8 #F07ADB` (our own values; the engine remaps them by brightness, the way Wesnoth does). They should cover 5–15% of opaque pixels. Never use these 4 colours for anything else, and never fill unused palette slots with them.
   - **Override:** pebbly's tip to "paint unused slots magenta" (`PEBBLY/rules/21-limited-and-platform-palettes.md:26`).
   - Check: [T `palette op=get`] for the share; [E] for placement.

### Light and shading

10. **Light.** Light comes from the upper-left of the canvas for every sprite and every frame. Highlights go on top and left-facing planes; the core shadow goes on the lower right. [E]
11. **No pillow shading.** Don't shade from the centre outward, and don't put a uniform dark rim inside the outline. Offset the highlight toward the light and put a crescent shadow on the far side (`PEBBLY/rules/92-generated-art-tells.md:46`, `PEBBLY/rules/02-shading-and-light.md`). [E]

### Outline and edges

12. **Selective outline.** Use one near-black, blue-purple outline colour (in the master palette, e.g. `#16121f`) on silhouette edges next to dark or mid materials. Next to light materials, use the darkest shade of that material. Draw no outlines between interior shapes. Never use `#000000`. [E; T `look op=ascii`]
    - Expect `validate`'s outline check to report a "note" or a "60–99% one colour" warning. That warning is a false positive for selective outlines (`PEBBLY/extension/ai-artist.lua:3818-3840`), so judge by eye. Follow `PEBBLY/rules/04-outlines-and-edges.md`.
13. **Binary alpha.** Every pixel is alpha 0 or 255. No soft brush and no translucent edge AA. Any AA uses opaque in-between colours inside the outline, at most half a run long (`PEBBLY/rules/12-anti-aliasing.md`). [V `antialiasing` in RGB mode]
14. **Clusters.** No orphan pixels and no confetti. Every colour cluster is at least 2 px except deliberate single-pixel highlights. [V `strays`; E for confetti per `PEBBLY/rules/11-clusters-and-noise.md`]
15. **Banding.** No straight colour boundary longer than a third of the sprite's width, and no doubled outlines. [V `banding`; E for doubles]

### Animation

16. **Required set and tags.** Every unit has these tagged animations: `breathing`, `attack`, `hit`, `death`, plus `run` if it moves. Use exactly these tag names. [V `animation` flags untagged frames; T `sprite_info` for the names]
17. **Durations vary.** Set per-frame durations from rules 18–21; never give a cycle uniform timing. [V `animation` note "Every frame has the same duration"]
    - **Override:** Duelyst's flat 80 ms. We follow Wesnoth and pebbly here (`PEBBLY/rules/40-timing-and-spacing.md`).
18. **breathing.** 4 frames in a loop of 1.0–1.6 s, e.g. 350/200/350/200 ms. Move the torso 1 px down, the head 1 frame later. The feet never move. [T `look op=filmstrip`; E]
    - This matches pebbly's 2–4 frame, 200–400 ms idle (`PEBBLY/rules/05-animation.md:12`). It is faster than pebbly's "calm" 300–500 ms (`PEBBLY/rules/41-idle-and-breathing.md:48`) because units on a board should look alert.
19. **attack.** 5–8 frames:
    - 1–2 anticipation frames pulling 2–4 px away from the target, 120–200 ms each.
    - One strike or smear frame of 50–80 ms.
    - One contact hold of 150–250 ms.
    - 1–2 recovery frames of 80–120 ms.

    The contact frame starts at 50–65% of the total duration (Duelyst ≈ 61%; Wesnoth hit at t=0 after ≈ 300 ms). Write that time in ms in the frame's user data or the export notes as `impact_ms`, so the engine syncs damage and FX to it.
    - **Override:** pebbly says player attacks get "no wind-up, or one frame of ≤ 50 ms" (`PEBBLY/rules/44-attacks-and-impacts.md:19`). That rule targets real-time input lag. hello1 attacks resolve in turns, so wind-ups are allowed and wanted (Wesnoth wiki: "err on the side of over-exaggerating … windups").
    - Check: [T `sprite_info` durations; E].
20. **Smear.** The strike frame may carry one smear or swoosh using the 2 lightest colours in the palette. Put it on a layer named `smear`, present on that frame only. [V with `expect.layerFrames`]
21. **hit and death.**
    - `hit`: 2–3 frames totalling 200–300 ms. The body recoils 1–3 px away from the attacker and returns to the base pose.
    - `death`: 6–10 frames at about 100 ms, with the last frame held ≥ 400 ms.
    - Don't draw a white or red flash in either; the engine does it. [T durations; E]
22. **run.** 6 frames at 80–100 ms, with contact poses held longer than passing poses (`PEBBLY/rules/42-walk-and-run.md`). [T; E]
23. **Volume.** Volume stays constant across frames. Move parts at different speeds, never only the "active element". Redraw a part rotated by more than 30° rather than sliding it. [E `look op=filmstrip`; `PEBBLY/rules/45-secondary-motion.md`]
24. **Sub-pixel motion.** For motion under 1 px in breathing, shift shade boundaries instead of moving the outline (`PEBBLY/rules/46-subpixel-animation.md`). [E]

### FX and export

25. **Nothing baked in.** No ground shadow, glow, bloom, light halo, particles, hit flash or team-coloured aura in the sprite; these are engine-side.
    - Check: [V `antialiasing` catches translucent shadows; E for opaque glow rings].
26. **Layer names.** `body` (plus rig parts per `PEBBLY/rules/06-layers-and-rigging.md`), `smear` (optional), `teamkey` (optional, keys only). No hidden draft layers in the saved file. [T `sprite_info`; V `layers`]
27. **Export.** One spritesheet per unit, JSON array with tags, 1x scale. The 4x preview PNG is for review only. Save before exporting. [V `export_readiness`]
28. **Comparison check.** Before calling a unit finished, put it next to at least one finished hello1 unit at 2x and check height, outline weight, light side and palette match. [E]

### Rules pebbly's `validate` cannot check

These always need an eye pass (pixel-critic or `look`): 3, 4, 7, 10, 11, 12, 18–19 (feel), 23, 24, 28.

---

## 6. How to load these rules into pebbly

### What pebbly supports, from its code

- **No project-rules hook.** The rulebook served as `rules://` is read only from the plugin's own `rules/` folder: `createServer` calls `loadRules(root)` with `root = packageRoot()` (`PEBBLY/src/server.ts:50-52`; `PEBBLY/src/lib/skills.ts:69-80`). There is no env var, config or project path for extra rules. Editing the plugin's `rules/` would be overwritten on update.
- **The brief is a chat message, not a file.** `aseprite:brief` drafts decisions from "what the user said plus the defaults" and says to "Record the brief in your working notes and hold to it". It also says matching an existing project's style "is almost always what they want" (`PEBBLY/skills/brief/SKILL.md`, steps 2–6). The studio skill passes "the document name and the brief" to the critic agent (`PEBBLY/skills/studio/SKILL.md:68`). The critic reviews "against the project rulebook" and the brief it is handed (`PEBBLY/agents/pixel-critic.md:3`, `:24-41`).
- **Palette files load from disk.** `palette op=load` reads `.gpl`, `.hex`, `.pal` or `.png` files (`PEBBLY/src/tools/palette.ts:64`, `:207-215`). The path goes to Aseprite (`palette.load`), so use an absolute path. That this is necessary is my inference, not checked.
- **AGENTS.md.** pebbly's installer can append a fixed section to `AGENTS.md` for clients without plugins (`PEBBLY/src/install.ts:170-197`, `PEBBLY/src/cli.ts:214-216`). It is its own text, not a project hook.

### So the mechanism is Claude Code's own project memory, plus the palette file, plus the brief

1. **Commit the rules.** Put section 5 (rules only, under 200 lines) in the hello1 repo as `.claude/rules/pixel-art.md`, with **no** `paths:` frontmatter.
   - Claude Code loads rules without `paths` "at launch with the same priority as `.claude/CLAUDE.md`" (https://code.claude.com/docs/en/memory, "Organize rules with .claude/rules/").
   - Don't use path-scoped rules here: they load only when Claude reads or edits a matching file, and pebbly works through MCP tools, not files (same page, "Path-specific rules").
2. **Commit the palette.** Add the palette file, e.g. `art/palette/hello1.gpl`. It doesn't exist yet: make it with the `aseprite:palette-smith` agent first, including the outline colour and the 4 team keys from rule 9.
3. **Pass the brief with each new sprite** (below), so `aseprite:brief` treats these values as decisions and only asks about the subject.
4. **Give the critic the brief.** When spawning `aseprite:pixel-critic`, pass it the brief and the rules file path. Whether plugin subagents receive project `.claude/rules` automatically is **unverified**.

### Ready-to-paste brief

Fill in the `<…>` parts.

```
hello1 unit sprite. Follow .claude/rules/pixel-art.md; it overrides pebbly defaults.
Subject: <one line: who/what, role, the one prop that identifies it>.
Canvas 80x80 RGB, transparent; body 40–56 px tall, centred, lowest foot pixel on row 61; faces right, side-on with slight top view.
Palette: palette op=load path=<ABSOLUTE PATH>/art/palette/hello1.gpl replace=true — no presets; ≤24 colours used; ramps 3–4 shades, shadows toward blue/purple, highlights warm.
Team-coloured parts (<which parts>) only in #4A0D3F #8A1A76 #C42DA8 #F07ADB, 5–15% of opaque pixels.
Light from upper-left. Selective outline: near-black blue-purple from the palette on dark/mid edges, darkest local shade on light edges, never #000000, none inside.
Alpha 0 or 255 only. No baked shadow, glow, flash or particles.
Animations (tags): breathing 4f 350/200/350/200; attack 5–8f (anticipation 120–200 ms, strike/smear 50–80, contact hold 150–250, recovery 80–120), contact at 50–65% of total — report impact_ms; hit 2–3f, 200–300 ms total; death 6–10f ~100 ms, last held ≥400; run 6f 80–100 if it moves.
Done = validate strict=true with only the expected selective-outline note, look preview at 2x next to an existing hello1 unit, and pixel-critic review against this brief.
```

### Suggested `.claude/CLAUDE.md` line (optional, if you prefer one entry point)

`Pixel art: follow @art/STYLE.md (hello1 style rules). It overrides aseprite-ai-artist defaults (PICO-8, 32×32, 3/4 view, no-wind-up attacks).`

The `@path` import syntax is from https://code.claude.com/docs/en/memory, "Import additional files". Use either this or step 1, not both: imported files still count toward context.

---

## 7. Licensing notes

- **Wesnoth.**
  - Code is GPL v2+ (`WESNOTH/COPYING`, `WESNOTH/README.md:79-84`). The README says "Most art and music is also licensed under the GNU GPL v2+, however new contributions are now licensed under the Creative Commons BY-SA v4.0."
  - The wiki: "All visual and audio assets are licensed under either the GNU GPL v2 or later, or the Creative Commons BY-SA 4.0", with CC BY-SA 4.0 "as of July 30th, 2017" (https://wiki.wesnoth.org/Wesnoth:Copyrights).
  - `WESNOTH/copyrights.csv` lists 396 files, almost all sounds and music (343 GPL v2+, 48 CC BY-SA 4.0, 5 CC0). It has **no** per-file entries for unit images, so the licence of any single sprite is unverified beyond the README and wiki statement.
- **What this means for Wesnoth.** Measuring canvas sizes, timings, colour counts and techniques, and writing our own rules, copies no expression, so hello1 isn't bound by GPL or CC BY-SA. Copying or tracing a sprite, its exact palette or the 19 magenta key values as an asset would make that asset GPL or CC BY-SA and would need attribution and share-alike. For a GPL asset, distribution terms would probably cover the combined work too (unverified legal reading, not advice). Our team keys (rule 9) are our own values. The *technique* of brightness-mapped exact-match keys isn't copyrightable expression (general principle, not legal advice).
- **Duelyst.** The repo and assets are CC0 1.0, but "No trademark or patent rights held by Affirmer are waived" (`DUELYST/LICENSE:104-105`; `EARLIER/` §4). Don't use its names, logo, faction or character names.
- **pebbly.** We use the plugin as a tool and quote its rule file names. Its licence is in `PEBBLY/LICENSE`, which I didn't read for this report (unverified).

---

## 8. Sources

### Local: Duelyst (`DUELYST/`)

`app/data/resources.js` (5136, 8045, frameDelay tally); `app/data/fx.js` (8-30, 5000-5040); `app/view/layers/game/GameLayer.js` (3380-3395); `app/view/nodes/cards/UnitNode.js` (240-256); `app/view/nodes/cards/EntityNode.js` (555-560, 584-586, 1961-1971); `app/common/config.js` (746, 751, 759); `app/view/nodes/fx/` (directory listing); `app/sdk/cards/factory/misc/tutorial.coffee` (73-75, 293-295); `app/sdk/**` (`attackDelay` / `attackReleaseDelay` tally); `app/resources/units/*.plist|png`; `LICENSE`. Everything else is via `EARLIER/`.

### Local: Wesnoth (`WESNOTH/`)

`data/core/team-colors.cfg` (2-80, 325-342); `data/game_config.cfg:45`; `src/color_range.cpp` (32-107); `src/image_modifications.cpp` (543-576); `src/units/drawer.cpp` (53-62, 86-87); `src/units/animation.cpp` (560-579, 790-814); `src/units/animation_component.cpp` (34-43); `src/game_config.cpp:55`; `data/core/macros/animation-utils.cfg` (82-130, 240-246); `data/core/units/humans/Loyalist_Fencer.cfg`; `data/core/units/humans/Loyalist_Spearman.cfg`; `data/core/units/**/*.cfg` (survey); `data/core/images/units/**` (survey); `data/core/images/portraits/humans/*.webp`; `README.md` (79-84); `COPYING`; `copyrights.csv`.

### Local: pebbly (`PEBBLY/`)

`src/server.ts` (1-60, 100-170); `src/lib/skills.ts` (41-80); `src/tools/craft.ts` (88-195); `src/tools/palette.ts` (60-120, 207-216, 287-321); `extension/ai-artist.lua` (3692-3960); `src/install.ts` (170-197); `skills/brief/SKILL.md`; `skills/new/SKILL.md:34`; `skills/palette/SKILL.md:16-20`; `skills/studio/SKILL.md`; `agents/pixel-critic.md`; `rules/01, 02, 03, 04, 05, 06, 11, 12, 14, 20, 21, 40, 41, 42, 44, 45, 46, 92` (`.md`).

### Scratch (`SCRATCH/`)

- Scripts: `scripts/pngread.py` (copied from the earlier `sprite_stats.py` reader), `scripts/anim_feel.py`, `scripts/attack_agg.py`, `scripts/wes_sprite_stats.py`, `scripts/partial_alpha.py`, `scripts/wes_anim_timing.py`, `scripts/upscale.py`.
- Outputs: `out/anim_feel.txt`, `out/attack_agg.txt`, `out/wes_sample.txt`, `out/wes_survey.txt`, `out/wes_unit_base2.txt`, `out/partial_alpha.txt`, `out/partial_alpha_all.txt`, `out/wes_anim_timing.txt`, `out/wes_4x.png`.

### Web (primary)

- https://wiki.wesnoth.org/Creating_Unit_Art
- https://wiki.wesnoth.org/Creating_Shadows_Under_Units
- https://wiki.wesnoth.org/Team_Color_Shifting
- https://wiki.wesnoth.org/Choosing_your_palette
- https://wiki.wesnoth.org/How_to_Anti-Alias_Sprite_Art
- https://wiki.wesnoth.org/Basic_Animation_Tutorial
- https://wiki.wesnoth.org/Art_Tutorials
- https://wiki.wesnoth.org/Wesnoth:Copyrights
- https://code.claude.com/docs/en/memory

### Not read (unverified)

- Wesnoth forum tutorials by core artists (e.g. http://forums.wesnoth.org/download/file.php?id=56284)
- Cure's Pixel Art Tutorial (linked from Creating_Unit_Art). pebbly's rules cite it (`PEBBLY/rules/12-anti-aliasing.md:253`).
- `WESNOTH/data/tools/TeamColorizer`
- Any Duelyst artist interview (none found, `EARLIER/` §5)
