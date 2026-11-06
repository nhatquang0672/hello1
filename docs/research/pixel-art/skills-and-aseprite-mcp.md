# Pixel art with Claude: skills, plugins, MCP servers, Aseprite

Researched 2026-11-06. Star counts and "last commit" dates come from the GitHub API (default-branch HEAD commit date) on that day [S1]. Numbers like [S3] point to the Sources list. "Unverified" means I could not confirm it from a primary source.

## Summary

- **Anthropic ships no pixel-art skill or plugin.** `anthropics/skills` has 19 skills and none is about pixel art. The nearest are `algorithmic-art`, `canvas-design` and `slack-gif-creator` [S2]. Neither the official marketplace (317 plugins) nor the community marketplace (2,283 plugins) has an Aseprite plugin. The community one does have two sprite *generators*: `sprite-gen` (Gemini) and `spritecook` (hosted MCP) [S3][S4].
- **There are at least 20 Aseprite MCP servers on GitHub** [S5]. Almost all are MIT and less than 8 months old. Only one is clearly established: **diivi/aseprite-mcp** (898★, last commit 2026-11-03, 123 tools, Python, headless `aseprite --batch --script`) [S6].
- **The servers work in one of two ways:** (a) **headless**: write a Lua script to a temp file and run `aseprite --batch --script` once per call (diivi, mattt, MalloyTheDev, willibrandon/pixel-mcp). (b) **live**: an Aseprite extension connects out over the Lua `WebSocket` client to a local bridge and edits the open window, with undo (with-pebbly, aseprite-mcp-pro, greenlogs, ZhangDongyang800 live mode) [S6][S7][S10][S11].
- **The best ready-made Claude Code plugin is `with-pebbly/aseprite-ai-artist`** (601★, MIT, v0.8.2). It is a Claude Code marketplace that bundles an 18-tool MCP server, 14 `/aseprite:*` skills, review subagents and a 62-file pixel-art rulebook. It drives the live window and has a `--headless` mode [S7]. **`willibrandon/pixel-plugin`** (414★) is the older equivalent, but its last commit was 2025-10-19 [S8].
- **Aseprite fully supports headless scripting.** `-b/--batch` runs without UI, `--script` runs Lua, `--script-param` feeds `app.params`, and `--sheet/--data/--split-tags` export spritesheets and JSON. On macOS the binary sits inside the .app bundle [S12][S13].
- **Cost: about $19.99** (Steam or itch, which includes a Steam key) [S14][S15]. **The free trial cannot save**: the source returns "Save operation is not supported in trial version" on every save op, so no MCP can write files with it [S16]. Compiling from source is free, but the EULA allows it only "for your own personal purpose" and forbids redistribution [S17].
- **Free alternatives:** LibreSprite (GPL-2.0) has `--batch --script` with **JavaScript**, not Lua. Its `main` branch now bundles an official `mcp-server.js` with 11 tools over HTTP, but that file is not in the v1.3 release [S18]. Pixelorama (MIT) has a community MCP with 40★ [S19]. The "Piskel MCP" re-implements Piskel's data model in TypeScript and does not drive Piskel itself [S20].
- **Recommendation:** buy Aseprite and add **diivi/aseprite-mcp** as a local-scope MCP server. That is one `git clone` plus one `claude mcp add`. Add the pebbly plugin later only if you want to watch Claude draw in the open window. Exact commands are under Recommendation below.

## Claude skills & plugins

| Name | Link | What it does (checked in code or manifest) | Stars / last commit | Licence | Verdict |
|---|---|---|---|---|---|
| Anthropic `skills` repo | github.com/anthropics/skills | 19 skills. None is pixel art. `slack-gif-creator` makes GIFs with Pillow, `algorithmic-art` uses p5.js [S2] | 187,911★ / 2026-10-28 | No repo-level licence (per-skill; unverified) | Nothing to use directly |
| Official marketplace `claude-plugins-official` | github.com/anthropics/claude-plugins-official | 317 plugins. Grep for pixel/sprite/aseprite/game finds only `unity`, `unreal-engine-skills-for-claude-code` and `figma` [S3] | 46,254★ / 2026-11-06 | Apache-2.0 | No pixel-art plugin |
| Community marketplace `claude-community` | github.com/anthropics/claude-plugins-community | 2,283 plugins. Pixel/sprite hits: `sprite-gen` (Gemini image gen) and `spritecook` (SpriteCook MCP: sprites, textures, animations) [S4] | n/a | n/a | Both are image generators (covered by the other agent); not checked further |
| **with-pebbly/aseprite-ai-artist** | github.com/with-pebbly/aseprite-ai-artist | Claude Code marketplace (`aseprite-ai-artist`) with plugin `aseprite`. `plugin.json` starts `node bin/aseprite-ai-artist.mjs serve` on 127.0.0.1:9931/9932. An Aseprite extension connects over WebSocket and edits the open document; each action is one undo step. 18 tools, skills `studio, brief, concept, draw, shade, animate, rig, palette, tileset, review, fix, export, new, submit`. `serve --headless` runs `aseprite -b`. SECURITY.md says "no telemetry, no network egress" [S7] | 601★ / 2026-10-22 (npm 0.8.2) | MIT | **Best plugin if you want live drawing plus craft guidance.** Needs Node 22.6+ and an extension installed into Aseprite |
| willibrandon/pixel-plugin | github.com/willibrandon/pixel-plugin | Claude Code plugin. `.mcp.json` runs a bundled prebuilt Go binary `bin/pixel-mcp` (from willibrandon/pixel-mcp, 158★, 17 open issues). It calls Aseprite with `--batch --script` [S8][S9]. Commands `/pixel-setup`, `/pixel-new`, `/pixel-export`, `/pixel-palette`, plus skills | 414★ / 2025-10-19 | MIT | Works in principle, but no commit in about a year and it ships opaque binaries. Second choice |
| Gamezxz/pixel-art-studio | github.com/Gamezxz/pixel-art-studio | Plain skill (`SKILL.md` + `scripts/pixelstudio.py`). Claude writes Pillow build scripts, renders, looks at the result and iterates. Exports PNG/GIF/sheet + Aseprite-style JSON [S21] | 105★ / 2026-07-11 | MIT | No Aseprite needed. Overlaps the Pillow pipeline the other agent covers |
| crsolver/claude-pxl | github.com/crsolver/claude-pxl | "Pixel art as code": `.px.js` DSL (palette + ASCII rows, mirror, outline), Node ≥ 22.18, no dependencies. Outputs PNG, sheet + JSON, GIF and a live viewer. Ships `SKILL.md` [S22] | 597★ / 2026-10-15 | MIT | Interesting no-Aseprite option. Created and last committed the same day, so maintenance is unproven |
| patch-gg/pixelfx | github.com/patch-gg/pixelfx | 244 code-defined pixel VFX (fire, slashes, impacts, lightning, projectiles) with sheet export, plus a Claude Code skill for authoring new ones. Method credited to Hugo Duprez [S23] | 5,440★ / 2026-10-31 (created 2026-10-30) | MIT | **Relevant to Duelyst-style card/attack VFX.** Very new; star count spiked within a week, so treat it with caution |
| 0x0funky/agent-sprite-forge | github.com/0x0funky/agent-sprite-forge | Skills that generate a master still with an image model, make image-to-video clips, then post-process into pixel sheets [S24] | 5,289★ / 2026-11-05 | MIT | Image-gen pipeline; out of scope here (other agent) |
| aldegad/sprite-gen | github.com/aldegad/sprite-gen | Sprite/atlas generation pipeline skill (Codex/Claude) [S1] | 3,760★ / 2026-11-06 | Apache-2.0 | Image-gen. Not inspected (unverified) |
| WhiteTowerAI/ohmygame `plugins/aseprite` | github.com/WhiteTowerAI/ohmygame | Contains `plugins/aseprite/skills/aseprite-assets/SKILL.md` inside a larger "AI game studio" app [S5] | 1,707★ / 2026-11-04 | Apache-2.0 | Not inspected (unverified) |
| omer-metin/skills-for-antigravity `pixel-art`, sickn33 `game-art` | GitHub | Generic prose "pixel art guidelines" skills from big skill collections [S5] | 193★ / 49,239★ | Apache-2.0 / MIT | Low value; generic advice |

## Aseprite MCP servers

All are on GitHub. "Batch" means one `aseprite --batch --script <tmp.lua>` process per call. "Live" means an Aseprite extension plus a WebSocket bridge.

| Name | Link | Tools / transport (checked) | Stars / last commit | Licence | Verdict |
|---|---|---|---|---|---|
| **diivi/aseprite-mcp** | github.com/diivi/aseprite-mcp | Python (FastMCP), stdio. **Batch**: `core/commands.py` builds `["--batch", ..., "--script", path]` and runs `ASEPRITE_PATH` with a 60 s timeout. 123 tools: canvas, layers, frames, tags, drawing incl. `draw_rows` (ASCII grid + legend), palettes/presets/quantize, dithering, outlines, tilemaps, slices, `export_spritesheet` (CLI `--sheet --sheet-type`), per-layer and per-tag export, `read_rows`, image previews returned to Claude (`view.py` → MCP `Image`), frame diff and lint, `run_lua_script` escape hatch. Python 3.13+, uv. Also offers Docker [S6] | 898★ / 2026-11-03; 3 open issues | MIT | **Recommended.** Most adopted and actively maintained. Headless, so it works without the window open |
| mattt/aseprite-mcp | github.com/mattt/aseprite-mcp | Go, stdio. **Only 2 tools**: `execute_lua` (batch) and `inspect_export` (returns a PNG as image content). Plus a Lua API reference and an `aseprite-lua` skill. Confines files to `--workspace` [S25] | 8★ / 2026-08-14 (only day of commits) | Apache-2.0 | Smallest context cost, and a clean design. One-day project, so maintenance risk. Needs Go 1.25 |
| MalloyTheDev/aseprite-mcp | github.com/MalloyTheDev/aseprite-mcp | Python, **batch** (`aseprite -b --script`). 165 tools incl. ramp shading, `assess_sprite`, `diff_sprites`, workflow tools for characters/tilesets, a batch op-runner, Godot export. Documents `claude mcp add aseprite -- uv --directory … run aseprite-mcp` [S26] | 25★ / 2026-10-20; 11 open issues | MIT | Very rich, small user base |
| ZhangDongyang800/Aseprite_MCP | github.com/ZhangDongyang800/Aseprite_MCP | Python (FastMCP). **Batch by default, optional live WebSocket mode.** Focuses on inspect-and-correct (`apply_operations`/`paint_grid`) [S27] | 17★ / 2026-11-02 | MIT | Interesting "eyes" angle; small |
| greenlogs/aseprite-mcp-live | github.com/greenlogs/aseprite-mcp-live | Python on PyPI (`aseprite-mcp`) + `.aseprite-extension`. **Live** (unsaved canvas, undo/redo) plus CLI batch. README is mostly Chinese and says it is **verified on Windows only** [S28] | 16★ / 2026-10-31 | MIT | macOS untested by the author |
| youichi-uda/aseprite-mcp-pro | github.com/youichi-uda/aseprite-mcp-pro | Repo contains **only the Lua extension**. The Node MCP server is sold separately ("Buy Me a Coffee… coming soon"). Live via WebSocket [S29] | 35★ / 2026-07-20 | MIT (extension only) | **Not usable as-is**: no server in the repo |
| with-pebbly/aseprite-ai-artist | see plugins table | Live (WebSocket bridge) + `--headless` batch; 18 tools [S7] | 601★ / 2026-10-22 | MIT | Best live option |
| willibrandon/pixel-mcp | github.com/willibrandon/pixel-mcp | Go, batch (`--batch … --script`), 40+ tools. Persists selections in `sprite.data` [S9] | 158★ / pushed 2025-10-18; 17 open issues | MIT | Stale |
| ext-sakamoro/AsepriteMCP | github.com/ext-sakamoro/AsepriteMCP | Python; palette, batch, Lua builder [S1] | 27★ / 2025-06-21 | MIT | Stale; not inspected |
| Others (giangdvdotdev, sloplink, rkdfx, ayigityol, shin2344234, Jofdiazdi, Vollkorn-Games, bachhoang0606, jcscocca…) | GitHub search "aseprite mcp" [S5] | Mix of batch and live | 1–13★ each | mostly MIT | Not inspected (unverified) |

Related: N7z/aseprite-live (10★, AGPL-3.0) streams an agent's Aseprite drawing to a browser and is linked from the diivi README [S6].

## Aseprite CLI/Lua capabilities & cost

**CLI (official docs [S12])**
- `-b, --batch`: "Runs Aseprite only to process command line options, then finishes" (no UI).
- `--script <file.lua>` and `--script-param name=value` (read through `app.params`). `--shell` gives an interactive Lua REPL.
- Export: `--save-as`, `--scale`, `--color-mode`, `--palette`, `--tag`, `--frame-range`, `--layer`, `--split-layers`, `--split-tags`.
- Spritesheets: `--sheet out.png`, `--sheet-type horizontal|vertical|rows|columns|packed`, `--data out.json`, `--format json-hash|json-array`, `--list-tags|--list-layers|--list-slices`.
- macOS binary: `/Applications/Aseprite.app/Contents/MacOS/aseprite`. Steam: `~/Library/Application Support/Steam/steamapps/common/Aseprite/Aseprite.app/Contents/MacOS/aseprite` [S12].

**Lua API (official docs [S13])**
- Objects: `Sprite, Layer, Frame, Cel, Image, Palette, Color, Tag, Slice, Tileset, Tile, Selection, Brush, Dialog, WebSocket, Timer, Plugin`, among others. Globals `app, app.command, app.fs, app.pixelColor, json`.
- `Image:drawPixel/putPixel/getPixel/drawImage/clear/resize/flip/saveAs` give direct pixel writes.
- `app.useTool` simulates real tool strokes (pencil, fill and so on). `app.transaction` groups edits into one undo step. `app.command.ExportSpriteSheet{type, textureFilename, dataFilename, dataFormat, splitLayers, splitTags, tag, trim, …}` [S30].
- `app.isUIAvailable` is false in `--batch` mode, so dialogs and alerts are unavailable there [S13b].
- `io.open` and `os.execute` "will ask for permissions to the user" [S13]. In batch mode with no UI this prompt may block or fail; that is unverified, and no MCP README mentions it.
- The Lua `WebSocket` is client-only, which is why live servers need a separate bridge process [S7 ARCHITECTURE.md].

**Headless on macOS:** the docs say `--batch` opens no UI and give the macOS binary path [S12]. diivi, mattt and MalloyTheDev all run headless batch and list macOS paths [S6][S25][S26]. I could not run it here. One forum report says `-b` returned before the script finished in a chained setup (unverified, older thread) [S31].

**Cost and licence**
- Official binaries: "$19.99/USD is the minimum pledge" for the 1.x series [S14]. The Steam price is $19.99 on third-party trackers (Steam store page not fetchable; unverified directly) [S15]. Buying on itch includes a Steam key [S15].
- **Trial:** "You CANNOT SAVE files with the trial version" [S32]. The source confirms this for every file save, including CLI/script `saveAs`. `ENABLE_TRIAL_MODE` → `"Save operation is not supported in trial version"` [S16]. **The trial is useless for MCP automation.**
- **Compile from source:** EULA §2(g) says "You may only compile and modify the source code … for your own personal purpose". §2(b) says no distribution to third parties [S17]. The default CMake options are `ENABLE_TRIAL_MODE off` and `ENABLE_SCRIPTING on`, so a self-built copy can save and run scripts [S33]. The macOS build needs Xcode 16.3, the macOS 15.4 SDK, CMake, Ninja and the prebuilt Skia `aseprite-m124`; `build.sh` automates it [S34]. Whether a self-build used to make a *commercial* game counts as "personal purpose" is ambiguous. The EULA does not address ownership of art made with Aseprite (unverified/legal question) [S17]. For $19.99, buying avoids the question.
- **LibreSprite** (GPL-2.0, free; latest release v1.3 on 2026-10-02, macOS arm64/x86_64 DMGs) [S18]. Scripting is **JavaScript** (QuickJS in `third_party/`, API in `SCRIPTING.md`), not Aseprite Lua, so no Aseprite MCP works with it. CLI has `--batch`, `--script`, `--set key=value`, `--sheet`, `--data`. `data/scripts/mcp-server.js` (added 2026-10-12, **not in v1.3**) is a built-in MCP server over Streamable HTTP on 127.0.0.1:9000–9099, no auth, 11 tools: `sprite_info, sprite_image, pixel_get, pixel_set, palette_get, palette_set, layers_list, sprite_save, sprite_save_as, command_info, run_command` [S18].

## Other tools

| Tool | What it is (checked) | Status | Verdict |
|---|---|---|---|
| abidoo22/Pixelorama-MCP | Node MCP for Pixelorama (free, MIT, Godot-based). 70+ tools, `draw_pixels_fast`, `eval_gdscript`, `capture_canvas_image`, Godot `.tres` export. Draws live in the Pixelorama window [S19] | 40★, MIT, pushed 2026-10-10. Transport details not inspected (unverified) | Best free-editor route if you won't buy Aseprite |
| yafeiaa/piskel-mcp-server | TypeScript MCP. Code shows its own `src/core/Piskel.ts` + pngjs/gifenc, so it **does not drive the Piskel app**. It is a standalone pixel canvas. Also depends on `@anthropic-ai/sdk` (purpose unverified) [S20] | 19★, Apache-2.0, 2026-07-29 | Fine toy; no advantage over a Pillow script |
| PixelLab MCP | Hosted HTTP MCP at `https://api.pixellab.ai/mcp` with a Bearer token. Tools `create_character` (4/8 directions), `animate_character`, `create_tileset` (Wang) and isometric tiles [S35] | Repo 47★, no licence, last commit 2025-08-10. Pricing not checked (unverified) | AI generation, paid account; overlaps the other agent's image-gen scope |
| light-vp/pixelart-mcp, comficker/pixel-art-mcp | Pillow-based pixel MCPs (draw/export; image→pixel with palette remap) [S5] | 7★ / 1★ | Not inspected (unverified) |

## Recommendation

**Smallest setup:** Aseprite (bought) + diivi/aseprite-mcp, local scope. It is headless, so Claude edits `.aseprite` files on disk and sees PNG previews through the MCP. You can open the same files in Aseprite and touch them up by hand. It is the only Aseprite MCP with real adoption and recent commits. If you prefer a tiny tool surface, mattt/aseprite-mcp (2 tools) works the same way, but it is a one-day project.

Steps (macOS; nothing was installed during this research):

```bash
# 1. Aseprite: buy on Steam or https://dacap.itch.io/aseprite (itch includes a Steam key). The trial cannot save.
#    Check that the CLI works (Steam path shown; for a /Applications install use /Applications/Aseprite.app/Contents/MacOS/aseprite):
ASE="$HOME/Library/Application Support/Steam/steamapps/common/Aseprite/Aseprite.app/Contents/MacOS/aseprite"
"$ASE" --version

# 2. uv (the server needs Python 3.13+, and uv fetches it)
brew install uv

# 3. Clone the server somewhere outside the game repo (path is your choice)
git clone https://github.com/diivi/aseprite-mcp.git ~/tools/aseprite-mcp

# 4. Register it with Claude Code, local scope (this project only; stored in ~/.claude.json)
cd ~/works/self/game-projects/hello1
claude mcp add --transport stdio --env ASEPRITE_PATH="$ASE" aseprite \
  -- uv --directory "$HOME/tools/aseprite-mcp" run -m aseprite_mcp

# 5. In Claude Code: /mcp shows "aseprite" connected. Smoke test prompt:
#    "Create a 32x32 sprite at art/test.aseprite, draw a red circle, export art/test.png and show me a preview."
```

The command shape comes from the Claude Code MCP docs (`claude mcp add [options] <name> -- <command> [args...]`, with `--transport stdio` placed between `--env` and the name) [S36]. The uv invocation matches diivi's README and `.mcp.json` (`uv --directory /path/to/repo run -m aseprite_mcp`) [S6]. `ASEPRITE_PATH` is read with `os.getenv` in `core/commands.py` [S6].

**Optional live-window upgrade** (watch Claude draw, get the skills and rulebook):

```bash
npx @pebbly/aseprite-ai-artist install-extension     # then quit and reopen Aseprite
claude plugin marketplace add with-pebbly/aseprite-ai-artist
claude plugin install aseprite@aseprite-ai-artist      # or in a session: /plugin install aseprite@aseprite-ai-artist
npx @pebbly/aseprite-ai-artist doctor
```
[S7][S37]. Do not run both Aseprite servers at once without reason: each adds tools to every turn.

**Free path, no Aseprite:** a Pillow skill such as Gamezxz/pixel-art-studio (`cp -r` into `.claude/skills/`; project skills live at `.claude/skills/<name>/SKILL.md` [S38]) or crsolver/claude-pxl. You lose the Aseprite editor for hand fixes.

## Sources

- [S1] GitHub GraphQL/REST API, repo metadata queried 2026-11-06 (`stargazerCount`, `defaultBranchRef.target.committedDate`, `licenseInfo`, open issues).
- [S2] https://github.com/anthropics/skills (tree `skills/`)
- [S3] https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json
- [S4] https://github.com/anthropics/claude-plugins-community/blob/main/.claude-plugin/marketplace.json
- [S5] `gh search repos "aseprite mcp"`, `gh search repos "pixel art mcp"`, `gh search code --filename SKILL.md "pixel art"` / `"aseprite"` (2026-11-06)
- [S6] https://github.com/diivi/aseprite-mcp: README, `aseprite_mcp/core/commands.py`, `tools/export.py`, `tools/view.py`, `.mcp.json`, issues
- [S7] https://github.com/with-pebbly/aseprite-ai-artist: README, `.claude-plugin/marketplace.json`, `.claude-plugin/plugin.json`, `docs/ARCHITECTURE.md`, `SECURITY.md`, `skills/`; https://www.npmjs.com/package/@pebbly/aseprite-ai-artist (v0.8.2)
- [S8] https://github.com/willibrandon/pixel-plugin: README, `.mcp.json`, tree
- [S9] https://github.com/willibrandon/pixel-mcp: README, `pkg/aseprite/client.go`
- [S10] https://github.com/youichi-uda/aseprite-mcp-pro (see S29)
- [S11] https://github.com/ZhangDongyang800/Aseprite_MCP (see S27)
- [S12] https://www.aseprite.org/docs/cli/
- [S13] https://www.aseprite.org/api/ ; [S13b] https://www.aseprite.org/api/app
- [S14] https://www.aseprite.org/faq/
- [S15] https://steambase.io/games/aseprite/price ; https://dacap.itch.io/aseprite (via web search; the Steam store page itself failed to fetch)
- [S16] https://github.com/aseprite/aseprite/blob/main/src/app/file/file.cpp (`ENABLE_TRIAL_MODE` block), https://github.com/aseprite/aseprite/blob/main/src/app/CMakeLists.txt
- [S17] https://github.com/aseprite/aseprite/blob/main/EULA.txt (§2(b), §2(g), §4)
- [S18] https://github.com/LibreSprite/LibreSprite: README, `SCRIPTING.md`, `src/app/app_options.cpp`, `data/scripts/mcp-server.js` (+ commit history), `third_party/quickjs-amalgam`, release v1.3
- [S19] https://github.com/abidoo22/Pixelorama-MCP
- [S20] https://github.com/yafeiaa/piskel-mcp-server: README, `package.json`, `src/` tree
- [S21] https://github.com/Gamezxz/pixel-art-studio
- [S22] https://github.com/crsolver/claude-pxl
- [S23] https://github.com/patch-gg/pixelfx ; https://www.spritefusion.com/blog/how-to-make-good-pixel-art-effects-with-claude
- [S24] https://github.com/0x0funky/agent-sprite-forge
- [S25] https://github.com/mattt/aseprite-mcp
- [S26] https://github.com/MalloyTheDev/aseprite-mcp
- [S27] https://github.com/ZhangDongyang800/Aseprite_MCP
- [S28] https://github.com/greenlogs/aseprite-mcp-live
- [S29] https://github.com/youichi-uda/aseprite-mcp-pro (README + tree: `extension/` only)
- [S30] https://www.aseprite.org/api/command/ExportSpriteSheet ; https://www.aseprite.org/api/image
- [S31] https://community.aseprite.org/t/batch-scripting-and-exportspritesheet/17395
- [S32] https://www.aseprite.org/trial/
- [S33] https://github.com/aseprite/aseprite/blob/main/CMakeLists.txt (lines 81–86)
- [S34] https://github.com/aseprite/aseprite/blob/main/INSTALL.md
- [S35] https://github.com/pixellab-code/pixellab-mcp
- [S36] https://code.claude.com/docs/en/mcp
- [S37] https://code.claude.com/docs/en/discover-plugins ; https://code.claude.com/docs/en/plugins/anthropic-marketplaces
- [S38] https://code.claude.com/docs/en/skills
