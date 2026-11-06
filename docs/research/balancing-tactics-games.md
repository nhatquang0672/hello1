# Balancing small deterministic tactics games

Research notes for hello1. Researched 2026-11-06. Claims are cited with a URL or a local path. Anything marked **(inference)** is my own reasoning and not sourced.

## 1. Summary: takeaways for hello1

1. Enemy count is the main difficulty lever. In ITB, one extra enemy turns "a fun challenge" into "completely impossible" ([Game Developer, IGF interview](https://www.gamedeveloper.com/game-platforms/road-to-the-igf-subset-games-i-into-the-breach-i-)), and spawn management is called "the most important part of Into the Breach balancing" ([ITB programmer's AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)).
2. Keep the AI dumb and greedy, and tune it through spawns and maps. ITB enemies score targets with a few fixed weights, never plan ahead, and ignore whether the board is solvable ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)).
3. Remove no-win states through map design, not AI logic: "It's easier to design the maps not to have those than to teach an AI to avoid them" ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)).
4. Clarity outranks cool. Subset cut weapons that testers couldn't understand from a tooltip ([Game Developer, UI article](https://www.gamedeveloper.com/design/-i-into-the-breach-i-dev-on-ui-design-sacrifice-cool-ideas-for-the-sake-of-clarity-every-time-)). Every part and combo must be readable on the board.
5. When unsure, err slightly easy: the game "remains fun even if it's slightly on the easy side" ([IGF interview](https://www.gamedeveloper.com/game-platforms/road-to-the-igf-subset-games-i-into-the-breach-i-)).
6. Log pick rate and win rate per part from the first playtest build. That data drove Slay the Spire's balancing, together with feel-based judgement ([Game Developer, StS data article](https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder)).
7. In single-player, strong combos are fine if they are rare and hard to assemble. Mega Crit's top rule is making infinites "really rare" ([StS data article](https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder)).
8. Make traits sidegrades, not upgrades. Wesnoth's `quick` trait gives +1 move and −5% HP (`wesnoth/data/core/macros/traits.cfg`). This fits the animals' single natural traits.
9. Use a resistance table per robot nature so that "nature × part" is a lookup, not special cases (Wesnoth `armoredfoot` takes 50% blade but 110% fire damage, `wesnoth/data/core/units.cfg`).
10. Give the leader a much larger HP pool than everything else. Duelyst generals have 25 HP against 2–10 for normal units (`duelyst/app/sdk/cards/factory/core/faction1.coffee`).

## 2. Into the Breach: how it was balanced

**Design frame.** Combat takes place on an 8×8 board with three mechs. Each map has an objective and a fixed number of turns, and losing the power grid ends the run ([Wikipedia](https://en.wikipedia.org/wiki/Into_the_Breach)). The power grid is the run's shared health. Damaged civilian buildings reduce it, so protecting buildings matters more than killing enemies ([Wikipedia](https://en.wikipedia.org/wiki/Into_the_Breach)). Subset wanted "every death [to feel] like your own fault" and built around "telegraphed enemy attacks as a core mechanic" ([IGF interview](https://www.gamedeveloper.com/game-platforms/road-to-the-igf-subset-games-i-into-the-breach-i-)). The turn counter and telegraphing were there to keep battles short, in contrast to hour-long XCOM fights ([Wikipedia](https://en.wikipedia.org/wiki/Into_the_Breach)).

**Perfect information as a constraint.** Justin Ma said the requirement that players understand every situation "restricted our game design options considerably". He also said: "we would sacrifice cool ideas for the sake of clarity every time". Weapons that playtesters still didn't understand "after reading three sentences" were cut, and animated tooltips were "the most important decision we made" ([Game Developer, UI article](https://www.gamedeveloper.com/design/-i-into-the-breach-i-dev-on-ui-design-sacrifice-cool-ideas-for-the-sake-of-clarity-every-time-)). During development the team cut roughly 60% of what they had planned, and called that their "eureka moment" ([NintendoSoup, summarising Kotaku](https://nintendosoup.com/?p=116329); search snippet only, not fetched).

**Enemy AI.** These details come from a Google Doc by the game's "sole programmer". The doc doesn't name the author; that it is Matthew Davis is my **(inference)** from his role.
- For each possible move and target, the AI scores hits with simple weights ("95% of them just use 5 pts, 5 pts, and -2 pts"), then picks randomly from among the best options.
- Lower difficulties let the AI pick "second best" options more often. Harder settings pick the optimal one more often.
- Enemies avoid danger tiles and map edges, and treat tiles that other enemies already target as dangerous. Apart from that, "Enemies don't care what other enemies are doing" and "Enemies never plan future moves". They also ignore the player's weapons and whether the position is solvable.
- Source: [AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic).

**Avoiding no-win states.** The same notes say things don't just get hard, "they get 'unsolvable' if everything isn't working correctly". The difference "is often decided by just adding 1 extra unit" against "your limited 3 actions". The defences are:
- Map design: avoid closed-in corners and terrain the player can't push enemies around.
- Spawn tuning: spawns adapt, so "kill a lot of units and spawns will increase to replace them".
- Iteration: "a long process of playtesting and iteration".
- The stated goal is to "keep every turn as interesting as possible, neither too easy nor too difficult".
- Source: [AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic).

A Vulture Q&A with Davis says most of the difficulty came from the spawn algorithm rather than from any single mission, and that tuning when and where enemies spawn took years. That claim comes from a mirror copy of the article; I couldn't fetch the original ([mirror](https://interakt.integrations.sell.do/top-news/into-the-breach-s-final-level-is-hard-without-breaking-the-rules.html)).

**Run structure.** A run is a sequence of islands with procedurally generated missions built from preset maps. Riskier islands give bigger rewards ([Wikipedia](https://en.wikipedia.org/wiki/Into_the_Breach)). Game Informer says the final island scales to how many islands you have completed ([Game Informer review](https://gameinformer.com/games/into_the_breach/b/pc/archive/2018/02/26/game-informer-review-into-the-breach.aspx); search snippet only).

**Residual randomness.** Grid damage has a small chance to miss. Reviewers put the starting value at 15%. Combat rolls otherwise have no randomness ([RPG Codex review](https://rpgcodex.net/article.php?id=10843); search snippet only). This miss chance is a failure buffer that only ever helps the player **(inference)**.

**Testing.** The GDC 2019 talk "Into the Breach Design Postmortem" (Davis) covers which features to cut and "how to decide on the correct difficulty" ([GDC Vault](https://gdcvault.com/play/1025772/-Into-the-Breach-Design), [80.lv](https://80.lv/articles/gdc-2019-an-inside-look-at-into-the-breach)). The recording is members-only and I didn't watch it. Isla Schanuel handled user testing ([Wikipedia](https://en.wikipedia.org/wiki/Into_the_Breach)).

## 3. Balancing builds and combos across a run

**Slay the Spire.** All of the following is from [Game Developer's coverage of Mega Crit](https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder) unless another source is given.

What Mega Crit tracked on a homemade metrics server:
- how often a card was picked when offered, and what it was picked over
- how often the card appeared in winning decks
- average damage taken from a specific enemy while using that card

How they used it:
- Giovannetti said hand-balancing the card pool was impossible: "there's no way we can intuitively do it all correctly".
- Data was not the only input: "The numbers are really useful but they're not telling us how things feel".
- Kaycee Yano said each look at the data should start from a specific question. The team's metrics grew from "three graphs" to "at least 90".
- They tuned in loops, changing a value and then checking player reactions ("keep tweaking the knobs").
- Example, Dual Wield: it was rarely picked, so they buffed it. Data showed the buff was too strong, so they narrowed it to Skills.
- Example, The Awakened One: data showed Power-heavy decks struggled against this boss, so they reduced its strength gain and raised its base damage.
- Their stated priority: "Going infinite is the number one thing we try to make really rare."

Talk summaries say Giovannetti's goal was not equal power for every card but making every card "reasonably viable". The same summaries say a single-player roguelike can tolerate strong combos if they are rare and hard to reach ([summary](https://summ-www.pages.dev/en-GB/summary/slay-the-spire-metrics-driven-design)). That is a third-party summary; the GDC talk itself ([GDC Vault](https://www.gdcvault.com/browse/gdc-19/play/1025731)) is login-gated.

**Keeping combos strong without one dominant build (inference):**
- Use pick rate against win rate as the detector. A part that is both high-pick and high-win is probably dominant. A low-pick, high-win part is underrated, so fix how it is presented. A low-pick, low-win part needs a buff.
- Offer a choice of three salvaged parts after each battle. That way no single build can be forced every run, the same way StS offers a choice of three card rewards.
- Tie each trigger part to a robot nature. A build then gets strong against some robots and weak against others, which works as a natural counter.

## 4. Techniques for deterministic grid games

Everything in this section is **(inference)** unless a citation is given.

- **Small integer scale.** ITB-scale numbers keep the math readable at a glance. Duelyst works with HP of 2–10 on normal units (`duelyst/app/sdk/cards/factory/core/faction1.coffee`). Wesnoth uses HP in the 30s, but its combat is random. With no RNG, a damage number of 1–4 against HP of 1–6 lets players count lethal in their heads.
- **Action economy.** Subset ties unsolvability to "your limited 3 actions" ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)). hello1 also has three bodies, so a turn can answer about three telegraphed threats, minus whatever a combo saves.
- **Threats per turn.** Start at threats ≤ actions. Use combos (one action answering two threats) as the way the player beats threats > actions in later fights.
- **Cooldowns as tempo.** Cooldowns replace the energy limit of card games. Give strong triggers longer cooldowns so a combo can fire about once per battle, not once per turn. That keeps it powerful without being degenerate, the same way StS keeps infinites rare.
- **Difficulty curve.** Scale spawn count and robot nature mix across battles, not raw HP. ITB put its tuning effort into spawns ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)), and the final island scales with progress ([Game Informer](https://gameinformer.com/games/into_the_breach/b/pc/archive/2018/02/26/game-informer-review-into-the-breach.aspx)).
- **Failure buffer.** A run-wide health pool lets the player trade a little damage for a clean solve, as ITB's grid does ([Wikipedia](https://en.wikipedia.org/wiki/Into_the_Breach)). In hello1 the human's HP plays this role. Give the human its own small buffer so that one mistake isn't instantly fatal.
- **Detecting broken combos.**
  1. Log every combo trigger, with the parts, robot nature and damage, plus per-battle turns-to-win and damage taken.
  2. Flag any combo that clears a battle in one turn, or that ends up in more than a set share of winning runs.
  3. Because the game is deterministic, a brute-force solver over one turn's moves can confirm whether a board is solvable and whether a combo is degenerate. Run it before human playtesters.

## 5. Notes from the local repos

**Wesnoth** (`/Users/quang.tran/works/self/game-projects/wesnoth`). I found no standalone balance doc in `doc/`. Balance lives in the data files.
- **Traits are small sidegrades** (`data/core/macros/traits.cfg`):
  - `strong`: +1 melee damage and +1 HP.
  - `quick`: +1 move, −5% HP.
  - `resilient`: +4 HP, plus 1 per level.
  - `intelligent`: −20% XP needed.
  - Useful model for the animals' single natural traits.
- **Resistances live in movetypes, not individual units** (`data/core/units.cfg`, `armoredfoot` at line 549). Values are damage taken as a percentage: blade=50, pierce=60, impact=90, fire=110, cold=100, arcane=90. A lookup like this suits "robot nature × part" (heavy, for example, takes less physical and more fire/electric damage) **(inference)**.
- **Cost tracks role, not raw stats.** The Spearman costs 14 with 36 HP, 5 moves and 7-3 melee. The Heavy Infantryman costs 19 with 38 HP, 4 moves and 11-2 melee (`data/core/units/humans/Loyalist_Spearman.cfg`, `Loyalist_Heavy_Infantryman.cfg`). The higher price pays for armor and damage per hit, while the unit gives up speed.

**Duelyst** (`/Users/quang.tran/works/self/game-projects/duelyst`). I found no balance notes in `docs/` (only infrastructure docs).
- **Stat curve.** I averaged attack + HP by mana cost across `app/sdk/cards/factory/core/*.coffee`. The parse only picked up cards that set atk, maxHP and manaCost in that order.

  | Mana | Average attack + HP |
  |---|---|
  | 1 | 3.2 |
  | 2 | 4.6 |
  | 3 | 5.6 |
  | 4 | 7.7 |
  | 5 | 9.6 |
  | 6 | 10.6 |
  | 7 | 15.3 |

  That is roughly 1.5–2 stat points per mana, with a jump at the top. Cards with abilities sit below the curve. For example, Second Sun is a 5-mana 0/8 and Elyx is a 6-mana 7/7 (`faction1.coffee` lines 393–415, 555–577).
- **The leader is a separate stat class.** Generals cost 0 mana and have 2 attack and 25 HP (`faction1.coffee` lines 85–119). Losing the general loses the game, which matches hello1's human.
- **Lesson (inference).** Put a "budget" on each part: base effect value, reduced by cooldown length and setup needs. Price abilities as stat points taken off the budget, the way Duelyst does.

## 6. Suggested starting balance checklist for hello1 (inference)

All of this is **(inference)**: starting values to playtest, not sourced facts.

- [ ] Robots have 1–4 HP and parts deal 1–3 damage. The human has 4–5 HP, and the animals have 3. No HP above about 6 in the first build.
- [ ] Turn 1 of battle 1 has 2 telegraphed threats against 3 actors. By the last battle, aim for about 4 threats, so at least one combo is needed every turn.
- [ ] Battles last 4–5 turns, with a survive-or-protect objective, ITB style.
- [ ] Spawns are the main difficulty dial. Tune spawn count and nature mix per battle first, AI cleverness last.
- [ ] Use greedy, non-planning AI with ITB-style scoring weights, picking randomly among the top scores. Make it pick "second best" more often on easy.
- [ ] Every map gets a solvability check: a brute-force solver confirms at least one line where the human takes no damage on turn 1.
- [ ] Map rule: no dead-end corners that robots can't be pushed out of.
- [ ] Use a nature resistance table (fuel, electric, heavy, light × damage types) with values only at 0.5×, 1× and 2×.
- [ ] Setup parts have cooldown 1–2 and trigger parts have cooldown 2–3, so a big combo lands about once per battle.
- [ ] Offer a choice of 3 salvaged parts after each battle, and no shop.
- [ ] Log each part's pick rate and win rate and each combo's fire count from the first playtest. Review any part whose win rate is far above the average.
- [ ] Any combo that clears a battle in one turn is either rare (needs 2 specific parts plus a matching nature) or gets nerfed.
- [ ] Every part effect must show as a tile preview. If it can't be previewed, cut it.
- [ ] When in doubt, make it slightly easier.

## 7. Unverified claims

- ITB grid power maxes at 7, grid defense starts at 15%, and missions usually last 4–5 turns. These numbers are widely cited by fans. I only saw the 15% figure in a review snippet, and the wiki pages refused automated fetches.
- "Every StS card and relic had its win rate and pick rate tracked across millions of runs." I found this on fan sites, not in a primary source.
- "Justin Ma: the AI picks randomly from the top of a damage list." This appears in secondary search summaries. The programmer's notes support the mechanism, but I couldn't trace this quote to Ma himself.
- The full content of the GDC talks by Davis (ITB) and Giovannetti (StS). Both are login-gated. Everything here comes from session abstracts and press coverage.
- Who wrote the ITB AI Google Doc. Attributing it to Davis is an inference from "sole programmer".

## 8. Sources

- Game Developer, "Road to the IGF: Subset Games' Into the Breach": https://www.gamedeveloper.com/game-platforms/road-to-the-igf-subset-games-i-into-the-breach-i-
- Game Developer, ITB UI and clarity article: https://www.gamedeveloper.com/design/-i-into-the-breach-i-dev-on-ui-design-sacrifice-cool-ideas-for-the-sake-of-clarity-every-time-
- ITB programmer's AI and balance notes (Google Doc): https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic
- GDC Vault, "Into the Breach Design Postmortem": https://gdcvault.com/play/1025772/-Into-the-Breach-Design
- 80.lv, GDC 2019 preview: https://80.lv/articles/gdc-2019-an-inside-look-at-into-the-breach
- Wikipedia, Into the Breach: https://en.wikipedia.org/wiki/Into_the_Breach
- Vulture Q&A with Davis (mirror copy): https://interakt.integrations.sell.do/top-news/into-the-breach-s-final-level-is-hard-without-breaking-the-rules.html
- NintendoSoup on the 60% cut: https://nintendosoup.com/?p=116329
- Game Informer review: https://gameinformer.com/games/into_the_breach/b/pc/archive/2018/02/26/game-informer-review-into-the-breach.aspx
- RPG Codex review: https://rpgcodex.net/article.php?id=10843
- Game Developer, "How Slay the Spire's devs use data to balance": https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder
- GDC Vault, "Slay the Spire: Metrics Driven Design and Balance": https://www.gdcvault.com/browse/gdc-19/play/1025731
- Third-party talk summary: https://summ-www.pages.dev/en-GB/summary/slay-the-spire-metrics-driven-design
- Local: `wesnoth/data/core/macros/traits.cfg`, `wesnoth/data/core/units.cfg`, `wesnoth/data/core/units/humans/Loyalist_Spearman.cfg`, `wesnoth/data/core/units/humans/Loyalist_Heavy_Infantryman.cfg`
- Local: `duelyst/app/sdk/cards/factory/core/faction1.coffee` and `duelyst/app/sdk/cards/factory/core/*.coffee`
