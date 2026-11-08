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

## Reports

- [duelyst-style-and-pipeline.md](duelyst-style-and-pipeline.md): Duelyst's sprite sizes, frames, palette and FX, measured from its repo, plus ways to make pixel art with Claude Code.
- [skills-and-aseprite-mcp.md](skills-and-aseprite-mcp.md): survey of Claude skills, plugins and Aseprite MCP servers.
- [best-skills-ranking.md](best-skills-ranking.md): ranking of those skills by quality.
- [style-guide-duelyst-wesnoth.md](style-guide-duelyst-wesnoth.md): what makes Duelyst's and Wesnoth's art work, and a draft rule list.

Scratch scripts and previews the reports cite under `/private/tmp/...` were not kept.
