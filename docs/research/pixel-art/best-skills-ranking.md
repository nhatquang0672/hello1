# Best Claude skills and plugins for pixel art: ranked for hello1

Researched 2026-11-06. This builds on `pixel-art-skills.md`, which covers the broad search, the Aseprite CLI and its cost. Here I read each shortlisted repo's SKILL.md, agent, command and hook files and the code they call. Repos were downloaded as tarballs into `scratchpad/repos/`. Nothing was installed or run. Repo metadata comes from the GitHub GraphQL API on the same day [S1]. "Unverified" means I could not confirm the claim from a primary source.

Target spec (Duelyst-like): 80–130 px canvases, characters 44–71 px tall, 7–24 colours, no partial alpha, no black outline, top light, purple/blue hue-shifted shadows, 2x nearest-neighbour render, idle/breathing/run/attack/hit/death animations at about 80 ms per frame.

## Summary

- **Best overall and best for animation: `with-pebbly/aseprite-ai-artist`.** It ships a 62-file pixel-art rulebook, 14 skills and 4 subagents, including a read-only `pixel-critic`. Its `validate` tool flags off-palette colours, semi-transparent pixels, stray pixels and uniform timing. `look` gives preview, ASCII, filmstrip and onion-skin views, and a PostToolUse hook tells Claude to look after every pixel edit [S2]. It needs Aseprite (about $19.99) and Node 22.6+.
- **Best free: `Gamezxz/pixel-art-studio`.** It uses Python and Pillow only. Its loop is "write build.py → render a preview at 10x → look → critique against a checklist". `stats()` counts `semi_alpha_px`, isolated pixels and near-duplicate colours. It exports a GIF and a spritesheet with JSON [S3]. Two flaws: one hard-coded path to fix, and a "learning" mode that edits the skill's own files.
- **Best for attack and hit VFX (an add-on, not a sprite tool): `patch-gg/pixelfx`.** It has 244 code-defined effects and a `check.mjs` linter, and exports sheets with JSON that carry wind-up, freeze and recovery timing. But it uses partial alpha **by design** ("truly translucent", glow baked in as alpha), which breaks the spec [S4]. It is also very new and gained stars very quickly.
- **`diivi/aseprite-mcp` is the strongest bare MCP server.** It has `view_sprite` image previews, `lint_sprite`, onion skin, `compare_frames`, a ramp tool with hue shift and an `animation_workflow_guide` tool. It is not a skill, and I found no partial-alpha check in it [S5]. Use it **instead of** pebbly if you want headless use and fewer moving parts, not alongside it.
- **`crsolver/claude-pxl`** is a neat free DSL with preview/ASCII and sheet/GIF/JSON output, but it teaches little craft. It has 2 commits, both on one day, and 598 stars [S6].
- **`willibrandon/pixel-plugin` is stale** (last commit 2025-10-19). It ships prebuilt binaries and has no image-preview loop: its read-back tool is `get_pixels` only [S7].
- **The paid generators (PixelLab MCP, agent-sprite-forge) are a poor fit.** PixelLab needs a subscription and its server code is not published [S8]. agent-sprite-forge's skill says a configured API key "is the owner's consent, so there is no per-call question", so it spends money without asking [S9].
- **The generic prose skills** (omer-metin, AbsolutelySkilled via zorai, deeptide `pixel-art-code`, curiositech `pixel-art-scaler`) give advice only, with no tooling and no review loop. Skip them [S10].

## Ranking

Craft score (0–5) measures how much real pixel-art method the skill files teach. Review loop is graded A–D: **A** means Claude sees an upscaled preview **and** a script checks palette and alpha; **B** means a preview plus some scripted checks, but not alpha; **C** means a preview only; **D** means no preview.

| # | Name | Link | Needs | Craft | Review loop | Animation | Maintenance (stars · last commit · commits · open issues · licence) | Verdict |
|---|---|---|---|---|---|---|---|---|
| 1 | aseprite-ai-artist (plugin `aseprite`) | github.com/with-pebbly/aseprite-ai-artist | Aseprite (paid), Node ≥ 22.6 | **5** | **A**: `validate` (palette, semi-alpha, strays, tags, uniform timing) + `look` preview/ascii/filmstrip/onion + `pixel-critic` agent + look-nudge hook | **Best**: per-frame durations, tags, rig layers, `animation-director` agent, filmstrip/onion, `validate expect` (mustNotOverlap), sheet + JSON `byTag`, GIF | 601 · 2026-10-22 · 160 · 0 · MIT | **Best overall.** Override its PICO-8 palette and slow idle timings in the brief |
| 2 | pixel-art-studio | github.com/Gamezxz/pixel-art-studio | Python 3 + Pillow (free) | 4 | **A**: `preview(scale=10)`, `save_silhouette`, `zoom`, `stats()` → `semi_alpha_px`, `isolated_px`, `near_duplicate_colors`, colour count; written checklist each pass | Good: frames, `set_duration`, tags, GIF, spritesheet + engine JSON, LPC sheet layout | 105 · 2026-07-11 · 21 · 2 · MIT | **Best free.** Fix the hard-coded path first |
| 3 | aseprite-mcp (MCP, no skill) | github.com/diivi/aseprite-mcp | Aseprite (paid), Python 3.13 + uv | 3 (inside tool docstrings) | **B**: `view_sprite` returns images, `lint_sprite`, `get_color_stats`, onion skin, `compare_frames`; no alpha check found | Strong tools: durations, tags, onion, `export_spritesheet`, per-tag export | 898 · 2026-11-03 · 43 · 3 · MIT | Best engine-only option; 123 tools add context cost on every turn |
| 4 | pixelfx (plugin) | github.com/patch-gg/pixelfx | Node ≥ 18; ffmpeg optional (free) | 4 for VFX, 1 for characters | **B**: contact-sheet PNG, `act.mjs` steps, `check.mjs` (seams, bounds, unseeded randomness, act timing); no palette or alpha check | VFX and attack "acts" → sheet + Aseprite-style JSON with fps and pivot | 5,451 · 2026-10-31 · 8 · 4 · MIT | Add-on for hit and attack VFX only; threshold its alpha before use |
| 5 | claude-pxl | github.com/crsolver/claude-pxl | Node ≥ 22.18 (free) | 2 | **C**: upscaled `_preview` PNG + `pxl ascii`; no alpha or palette lint | `anim()` with per-frame `ms`, strip + JSON with frameTags + GIF, `sheet()` | 598 · 2026-10-15 · 2 · 1 · MIT | Fine free toy; unproven maintenance |
| 6 | pixel-plugin | github.com/willibrandon/pixel-plugin | Aseprite (paid); bundled Go binaries | 3 | **D**: `get_pixels` only; no image preview | Frames, durations, tags, export | 414 · 2025-10-19 · 49 · 1 · MIT | Stale; skip |
| 7 | aseprite-lua skill (with mattt/aseprite-mcp) | github.com/mattt/aseprite-mcp | Aseprite (paid), Go 1.25 | 1 | **C**: `inspect_export` returns a PNG | Lua API references only | 8 · 2026-08-14 · 2 · 0 · Apache-2.0 | Clean, small; teaches API, not craft |
| 8 | PixelLab MCP | github.com/pixellab-code/pixellab-mcp | Paid PixelLab subscription (hosted) | n/a (image model) | None in the skill | `animate_character`, 4/8 directions | 47 · 2025-08-10 · 17 · 8 · **no licence**; repo is a README only | Generator, not a craft tool; no style control for this spec (unverified) |
| 9 | agent-sprite-forge (plugin) | github.com/0x0funky/agent-sprite-forge | Paid image/video API keys or Codex/Grok CLI; numpy/scipy/Pillow, ffmpeg | n/a (image → video → frames) | Has QA scripts (not inspected) | Video-to-frames per action | 5,291 · 2026-11-05 · 247 · 7 · MIT | Out of scope; spends money silently |
| 10 | Generic prose skills | see S10 | none | 1–2 | none | prose | various | Skip |

## Per-skill notes

### 1. with-pebbly/aseprite-ai-artist

- **Craft.** It teaches real method: "Most sprites want **8–32 colours total**. Per material, 3–5 steps" and "**Shadows** rotate toward the ambient — in practice, **toward blue/purple**" [rules/01](https://github.com/with-pebbly/aseprite-ai-artist/blob/main/rules/01-palette-and-color.md), [rules/02](https://github.com/with-pebbly/aseprite-ai-artist/blob/main/rules/02-shading-and-light.md). The outline rules give a table with "None", "Selective" and "Coloured" options and say "Pure black outlines on everything read as harsh" [rules/04](https://github.com/with-pebbly/aseprite-ai-artist/blob/main/rules/04-outlines-and-edges.md). On alpha: "Never leave semi-transparent pixels in pixel art … `validate` flags them" [rules/02]. Further files cover anti-aliasing, clusters, idle and breathing, attacks and impacts, and "generated-art tells" (`rules/12`, `41`, `44`, `92`).
- **Review loop.** `review/SKILL.md` opens with "'Done' is a claim. This is how you earn it." It starts with a cold read and then runs `validate` [skills/review](https://github.com/with-pebbly/aseprite-ai-artist/blob/main/skills/review/SKILL.md). The `pixel-critic` agent is read-only (`disallowedTools: Write, Edit`) and puts its cold read "before you use the brief" [agents/pixel-critic.md](https://github.com/with-pebbly/aseprite-ai-artist/blob/main/agents/pixel-critic.md). Lint is in code: `craft.ts` checks "off-palette colours, orphan/stray pixels, broken or doubled outlines, banding, semi-transparent … pixels, … untagged frames and uniform frame timing" (`src/tools/craft.ts:93`). The hook `look-nudge.mjs` injects "Call `look` before deciding whether it worked" once per session.
- **Animation.** "a vision model reads only the first frame of a GIF, so a filmstrip is the only way to actually see the motion" [rules/05](https://github.com/with-pebbly/aseprite-ai-artist/blob/main/rules/05-animation.md). Export uses `export op="spritesheet" … byTag=true padding=1` [skills/export](https://github.com/with-pebbly/aseprite-ai-artist/blob/main/skills/export/SKILL.md).
- **Fit gaps.** The default palette is "**PICO-8** (16 colours)" (rules/01), and the timing table says "Idle (breathing) 2–4 frames, 200–400ms each" (rules/05). Both conflict with the spec's custom 7–24 colour palette and 80 ms frames, so state those in the `/aseprite:brief`. Pick the "None" or "Coloured" outline style explicitly.
- **Runtime.** Dependencies are only `@modelcontextprotocol/sdk`, `ws` and `zod` (package.json). A grep of `src/` found no `fetch(` or http URLs. SECURITY.md says there is no telemetry ([S2], from the earlier survey). Arbitrary Lua is gated: "`lua.run` refuses unless the Aseprite process carries `ASEPRITE_AI_ALLOW_LUA=1`" (docs/INSTALL.md).

### 2. Gamezxz/pixel-art-studio

- **Craft.** "That see→critique→fix loop is the entire quality engine — never skip it." Make "Minimum 2–3 loop passes before showing the user" [SKILL.md](https://github.com/Gamezxz/pixel-art-studio/blob/main/SKILL.md). The `ramp(base, steps, hue_shift)` call builds hue-shifted ramps. The checklist asks for "no pillow shading", "ramps are hue-shifted (shadows cooler, highlights warmer)", "semi_alpha_px == 0" and "body bob ≤2px; head/secondary parts lag 1 frame" [references/validations.md](https://github.com/Gamezxz/pixel-art-studio/blob/main/references/validations.md).
- **Scripted checks.** `stats()` really counts `0 < a < 255` pixels, isolated pixels and near-duplicate colours (`scripts/pixelstudio.py:899–933`).
- **Extra.** `pixelpipe.py` cleans images from image models: it recovers the grid, hardens alpha, locks the palette and caps colours with `--max-colors 24`. This pairs well with any generator.
- **Flaws.** (a) The template hard-codes `sys.path.insert(0, "/Users/game/Projects/.claude/skills/pixel-art-studio/scripts")` (SKILL.md line 26), which you must edit. (b) Learning mode tells Claude to "write durable knowledge only inside" the skill directory, so the skill rewrites itself. (c) "then `open` the output files (user preference)" is the author's own preference leaking into the skill. (d) The repo was built over 2 days and has had no commit since July.

### 3. diivi/aseprite-mcp

- `animation_workflow_guide` returns concrete guidance. Examples: "Vary timing … 80-120 ms for normal frames, 150-250 ms for the anticipation hold, 50-80 ms for the strike". "For very fast moves, use one smear frame". "get_color_stats on each frame must report the same unique_colors count" [tools/guide.py](https://github.com/diivi/aseprite-mcp/blob/main/aseprite_mcp/tools/guide.py). `generate_color_ramp(hue_shift_degrees=20)` is in tools/palette.py:268.
- `lint_sprite` reports detached groups, lone pixels and orphan pixels (tools/analysis.py:536). **I found no partial-alpha check** (a grep of analysis.py and quality.py for alpha found none; unverified whether another tool covers it).
- It has no skill or agent files, so the craft lives only in tool docstrings.

### 4. patch-gg/pixelfx

- The method follows Hugo Duprez: "One-shot prompts make mediocre effects. Drafts you render, look at and refine make good ones." The skill asks for "Three rounds minimum" and a `check.mjs` "until it passes clean" [skills/pixelfx/SKILL.md](https://github.com/patch-gg/pixelfx/blob/main/skills/pixelfx/SKILL.md). Its fix table is good craft, for example "floaty, no punch → fast in, slow out …, a 1 to 2 frame white hero frame".
- **Spec conflict.** It draws "color (pixels: opaque, dithered or truly translucent)" and "Glow is baked into the sheet as color with alpha". You would need a per-effect alpha threshold, or the Bayer dithered-alpha path it mentions for ports.
- The skill tells Claude to `git clone https://github.com/patch-gg/pixelfx` next to the project.

### 5. crsolver/claude-pxl

- "**Look at it**: read `out/_preview/<name>.png` (upscaled …). For exact pixel fixes use `PXL ascii`" [SKILL.md](https://github.com/crsolver/claude-pxl/blob/main/SKILL.md). Its craft is mostly efficiency habits ("Write half of anything symmetric (`mx`) and let `outline` draw outlines") plus a good "No filler noise" section. It has no hue-shift or AA guidance. `glow` uses a screen blend that can create off-palette colours unless you pass `ramp` (reference.md, Glow section).

### 6. willibrandon/pixel-plugin

- It says the right words ("Pure black shadows (use dark hue-shifted colors instead)", skills/pixel-art-professional/SKILL.md:122). But the animator skill's `allowed-tools` contains no preview tool, and pixel-mcp's read-back tool is "`get_pixels` … for verification" [S7]. It ships `bin/pixel-mcp-darwin-arm64` and other prebuilt binaries.

### 7–10. Brief notes

- **mattt aseprite-lua**: "Write one script that builds the whole sprite, then look at the result", with an honest "Trust and scope" section. It is API reference only.
- **PixelLab**: the README says "Copyright © 2025 PixelLab. All rights reserved." The server is hosted at `https://api.pixellab.ai/mcp` with a Bearer token. Its docs list "active subscription" as a requirement. Third-party pricing reports conflict, roughly $9–50 per month (unverified) [S8].
- **agent-sprite-forge**: "HD is the default … The pixel finish … is an option". Characters are animated by image-to-video [S9].
- **Prose skills**: deeptide's skill draws 12×12 sprites with `K: '#11131a', // outline` black outlines, the opposite of the spec. The curiositech scaler targets hq2x/xBR, which conflicts with the spec's nearest-neighbour render [S10].

## Red flags

| Repo | Flag | Severity |
|---|---|---|
| pixelfx | Created 2026-10-30, 5,451★ by 2026-11-06, 8 commits. It went viral with Duprez's blog post (that cause is unverified). The stargazer-timestamp API returned 404 here, so I could not audit the stars. | Medium (popularity ≠ quality) |
| claude-pxl | 598★ on 2 commits made the same day (2026-10-15); same audit limit as pixelfx | Medium |
| pixel-art-studio | Hard-coded absolute path; the skill modifies its own files; leaked author preference | Low–medium |
| aseprite-ai-artist | The first run executes `npm ci --ignore-scripts` + `tsc` inside the plugin dir (bin/aseprite-ai-artist.mjs, launcher.mjs). It is lockfile-pinned and blocks install scripts, which is good practice, but it still downloads packages on first start. Two hooks inject context text; I read both, and they are benign status and "look" reminders. `apps/web` has an analytics module, but it belongs to the gallery website, not the plugin runtime. | Low |
| pixel-plugin | Opaque prebuilt Go binaries; no commit in about a year | Medium |
| PixelLab | No server code, no licence, paid hosted service, art goes through a third party | Medium |
| agent-sprite-forge | "The configured key is the owner's consent, so there is no per-call question": paid generation with no confirmation | High for cost |
| All | None uses curl-pipe-sh installs. I found no prompt-injection-style text in the SKILL, agent or hook files I read. Pebbly's "Use PROACTIVELY" agent wording is a normal trigger hint. | — |

## Recommendation & install

**If you buy Aseprite (recommended): pebbly.** It is the only one that combines a deep rulebook, scripted palette and alpha validation, a critic agent and filmstrip review for animation. Put the Duelyst spec into the brief so its defaults don't win: a custom 7–24 colour palette, no black outline, top light, 80 ms frames, and 2x nearest-neighbour previews.

```bash
# from the pebbly README
npx @pebbly/aseprite-ai-artist install-extension      # then restart Aseprite
claude plugin marketplace add with-pebbly/aseprite-ai-artist
claude plugin install aseprite@aseprite-ai-artist --scope local   # --scope per Claude Code docs; or in-session: /plugin install aseprite@aseprite-ai-artist
npx @pebbly/aseprite-ai-artist doctor
# no window: set ASEPRITE_AI_HEADLESS=1 and ASEPRITE_PATH=<aseprite binary> in the shell that starts claude (docs/INSTALL.md)
```

Do not also add diivi/aseprite-mcp: both servers would add tools to every turn.

**Free path: Gamezxz/pixel-art-studio.**

```bash
# from its README; install as a project skill
git clone https://github.com/Gamezxz/pixel-art-studio.git
cp -r pixel-art-studio <hello1>/.claude/skills/pixel-art-studio
pip3 install pillow
# then edit SKILL.md line 26 so sys.path points at <hello1>/.claude/skills/pixel-art-studio/scripts
```

**Optional, for attack and hit VFX:**

```bash
claude plugin marketplace add patch-gg/pixelfx
claude plugin install pixelfx@pixelfx --scope local
```

Expect to threshold its alpha to meet the no-partial-alpha rule.

The command forms come from the Claude Code docs: `claude plugin marketplace add owner/repo`, then `claude plugin install <plugin>@<marketplace>`. Shell installs default to user scope, and "Pass `--scope project` or `--scope local` to change it" [S11]. Project skills live in `.claude/skills/<name>/SKILL.md` [S12].

## Sources

- [S1] GitHub GraphQL API, 2026-11-06 (stargazerCount, pushedAt, commit count, open issues, licence) for all repos above; `gh search code --filename SKILL.md` for "pixel art", "spritesheet" and "hue shift".
- [S2] https://github.com/with-pebbly/aseprite-ai-artist: `.claude-plugin/plugin.json`, `rules/01,02,04,05,07`, `skills/review`, `skills/export`, `agents/pixel-critic.md`, `hooks/*.mjs`, `bin/*.mjs`, `src/tools/craft.ts`, `src/tools/look.ts`, `package.json`, `README.md`, `docs/INSTALL.md`
- [S3] https://github.com/Gamezxz/pixel-art-studio: `SKILL.md`, `references/validations.md`, `scripts/pixelstudio.py`, `README.md`
- [S4] https://github.com/patch-gg/pixelfx: `skills/pixelfx/SKILL.md`, `.claude-plugin/*.json`, `README.md`, `tools/`; https://www.spritefusion.com/blog/how-to-make-good-pixel-art-effects-with-claude
- [S5] https://github.com/diivi/aseprite-mcp: `aseprite_mcp/tools/guide.py`, `analysis.py`, `palette.py`, `quality.py`, `view.py`
- [S6] https://github.com/crsolver/claude-pxl: `SKILL.md`, `reference.md`, `README.md`
- [S7] https://github.com/willibrandon/pixel-plugin: `.mcp.json`, `bin/`, `skills/*/SKILL.md`, `README.md`; https://github.com/willibrandon/pixel-mcp README
- [S8] https://github.com/pixellab-code/pixellab-mcp README; https://www.pixellab.ai/docs/ways-to-use-pixellab (via search snippet); https://app.cinevva.com/guides/ai-pixel-art-generators and https://devtoollab.com/ai-tools/pixellab-ai (conflicting pricing, unverified)
- [S9] https://github.com/0x0funky/agent-sprite-forge: `skills/generate2dsprite/SKILL.md`, README
- [S10] https://github.com/paean-ai/deeptide/blob/main/skills/pixel-art-code/SKILL.md ; https://github.com/mkurman/zorai/tree/main/skills/nontechnical/absolutelyskilled/pixel-art-sprites ; https://github.com/curiositech/some_claude_skills/tree/main/.claude/skills/pixel-art-scaler ; https://github.com/omer-metin/skills-for-antigravity/tree/main/skills/pixel-art
- [S11] https://code.claude.com/docs/en/discover-plugins
- [S12] https://code.claude.com/docs/en/skills (cited from the earlier survey)
