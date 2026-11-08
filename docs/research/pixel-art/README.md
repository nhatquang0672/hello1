# Pixel art research

Research for hello1's art, done 2026-11-06. The decisions below override anything in the reports that disagrees (for example, the style guide's 80×80 side-on canvas and magenta team keys).

## Decided

| | |
|---|---|
| Style | Pure pixel art, learning from Duelyst and Wesnoth. No assets, names or characters copied. |
| View | Isometric. The game grid stays square, drawn as diamonds. |
| Light | From above, upper-left bias. Units get only a slight left/right difference, because they are mirrored. |
| Animation | Wesnoth-style: few frames, each with its own duration. |
| Team marker | Drawn by game code (a coloured ring under the unit), not painted into sprites. |
| Tools | Aseprite + the `aseprite-ai-artist` Claude Code plugin, live mode. |

## Proposed, not confirmed

- Platform: web first, sized for a phone browser in landscape.
- Sizes: 48×24 isometric tiles, 64×64 units, 96×96 bosses.

## Setup: connect Aseprite to Claude Code (once)

1. Install Aseprite (bought; the trial can't save, so automation fails).
2. Install the Aseprite extension, then quit and reopen Aseprite:
   ```bash
   npx @pebbly/aseprite-ai-artist install-extension
   ```
3. Install the Claude Code plugin:
   ```bash
   claude plugin marketplace add with-pebbly/aseprite-ai-artist
   claude plugin install aseprite@aseprite-ai-artist
   ```
4. Check the setup (needs Node 22.6+):
   ```bash
   npx @pebbly/aseprite-ai-artist doctor
   ```
5. Start a new Claude Code session in this repo. The plugin starts a local bridge on `127.0.0.1:9931/9932`, and the extension in the open Aseprite window connects to it.
6. Ask Claude to run `preflight`. It must report `ready: true`. If it says the bridge is up but Aseprite is not attached, open Aseprite (or redo step 2 and restart Aseprite).

Don't also install another Aseprite MCP server (such as diivi/aseprite-mcp): each one adds tools to every turn.

## Using it to make art

- Keep Aseprite open before asking for art. Claude draws in the open window, and each action is one undo step.
- Start with `/aseprite:studio <request>`. It picks the other workflows and runs them in order: `brief` (size, palette, view, light) → `concept` (optional reference image) → `draw` → `shade` → `review` → `export`.
- Other commands: `/aseprite:new`, `palette`, `rig` and `animate` (for units that move), `tileset` (for terrain), `fix` (to change existing art).
- Always give the decisions above in the request: isometric view, light from the upper left, 48×24 tiles, 64×64 units, no team colours in the sprite.
- Nothing reaches disk until you save in Aseprite or Claude runs `export`. Ask for exports with tags if a sprite is animated.
- Headless mode (no window, for batches of files) exists. Ask for it explicitly; Claude won't switch to it on its own.

## Reports

- [duelyst-style-and-pipeline.md](duelyst-style-and-pipeline.md): Duelyst's sprite sizes, frames, palette and FX, measured from its repo, plus ways to make pixel art with Claude Code.
- [skills-and-aseprite-mcp.md](skills-and-aseprite-mcp.md): survey of Claude skills, plugins and Aseprite MCP servers.
- [best-skills-ranking.md](best-skills-ranking.md): ranking of those skills by quality.
- [style-guide-duelyst-wesnoth.md](style-guide-duelyst-wesnoth.md): what makes Duelyst's and Wesnoth's art work, and a draft rule list.

Scratch scripts and previews the reports cite under `/private/tmp/...` were not kept.
