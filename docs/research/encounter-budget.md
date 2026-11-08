# Encounter budget: balancing the team against the robots

Research notes for hello1. Researched 2026-11-08. Every number has a local path or URL. **(inference)** marks my own reasoning. This builds on `balancing-tactics-games.md`, `enemy-ai.md`, `hp-scale.md` and `animal-roster.md` and doesn't repeat them. Local paths are relative to `game-projects/`. ITB paths point to the third-party script copy https://github.com/2147211690/LearnBreach (`scripts/...`), as in the earlier notes.

Design update used here: a run starts with the human + Dog. More teammates join later.

## Key findings

1. **Unit count matters more than total points.** D&D 5e (2014) multiplies monster XP by ×1.5 for 2 monsters and ×2 for 3–6, because "the more monsters there are, the more attack rolls you're making" ([Basic Rules](https://www.dndbeyond.com/sources/basic-rules/building-combat-encounters)). Parties of fewer than three use the next-higher multiplier (same page).
2. **Budgets scale linearly with party size.** PF2e adds 20 XP to an 80 XP moderate budget for each character beyond four (25% each) and subtracts it for each one missing ([AoN, GM Core p. 75](https://2e.aonprd.com/Rules.aspx?ID=2716)). D&D 2024 multiplies a per-character budget by party size ([DM's Toolbox](https://www.dndbeyond.com/sources/dnd/free-rules/dms-toolbox)).
3. **D&D 2024 dropped the multiplier and replaced it with a rule.** "If your encounter includes more than two creatures per character, include fragile creatures" ([DM's Toolbox](https://www.dndbeyond.com/sources/dnd/free-rules/dms-toolbox)).
4. **ITB scales by progress, not by how strong your squad is.** Starting enemies are {3,3,4,4} by sector. Normal spawns 2 then 3 per turn, with at most 6 enemies on the board. The Alpha share rises from 0/5 to 4/5 on Normal (`missions/missions.lua` lines 86, 998–1004; `spawner.lua` lines 46–70).
5. **ITB's only rubber band works on enemy count.** If 2 or fewer enemies remain, it spawns 1 extra (not on Easy). Spawns never push the board past the cap (`missions/missions.lua` lines 957–962).
6. **Wesnoth campaigns scale the enemy and leave the player alone.** In HttT "Blackwater Port", enemy starting gold is 135/250/365/475 by difficulty and enemy income 8/24/40/56 per turn. Konrad gets 75 gold and +0 income on every difficulty (`Heir_To_The_Throne/scenarios/03_Blackwater_Port.cfg` lines 46–174; `utils/unified_characters.cfg` lines 11–16).
7. **Duelyst prices bodies at about 3 stat points per mana and charges for keywords.** A vanilla 2-drop has ~6 stats and a 6-drop ~16. Single-keyword cards give up about 2 stats for Flying or Provoke, ~4 for Ranged or Forcefield, and 6–10 for Rush (my tally, section 1).
8. **Stat formulas explain only part of the price.** HP, moves and damage explain 36% of the variance in Wesnoth level-1 unit costs (R² = 0.36, my regression). Abilities and resistances make up the rest.
9. **Valve's Director changes pacing, not difficulty.** "Algorithm adjusts pacing, not difficulty." "Amplitude (difficulty) is not changed, frequency (pacing) is" (Booth, *The AI Systems of Left 4 Dead*, via [transcript](https://www.readkong.com/page/the-ai-systems-of-left-4-dead-michael-booth-valve-9664541)).
10. **Recommendation (inference):** budget against an *expected* team for the stage. Pass changes in unit count through at 100% and part upgrades at 50%. Add a per-turn count cap and a count multiplier. Then let a solver and playtest logs replace the guesses.

## 1. Duelyst: stat budget per mana

Method: I parsed every `new Unit` block in `duelyst/app/sdk/cards/factory/*/*.coffee` and dropped generals, tokens and `misc/` (tutorial and boss) cards, leaving 487 minions. "Vanilla" means no modifiers and no follow-ups. The parse script is in my scratchpad, not the repo.

| Mana | Vanilla atk+HP (cards) | All minions, mean atk+HP |
|---|---|---|
| 1 | 5, 4 (Squire, Komodo Charger) | 3.5 |
| 2 | 7, 5, 6 (Phalanxar, Skyrock Golem, Carcynus) | 4.4 |
| 3 | 7, 10 (Bloodshard Golem, Terradon) | 5.9 |
| 4 | 10, 10, 13 (Thorn Needler, Hailstone, Razorcrag) | 7.4 |
| 5 | 13 (Brightmoss Golem) | 9.3 |
| 6 | 16, 16, 15 | 10.9 |
| 7 | 20 (Dragonebone Golem) | 13.9 |

Examples: Komodo Charger 1/3 for 1 (`core/neutral.coffee` line 142), Skyrock 3/2 for 2 (873), Stormmetal 8/8 for 6 (964), Dragonebone 10/10 for 7 (933). Vanilla bodies gain about 3 stats per mana. Across all cards the slope is 1.8 per mana, because abilities eat the rest.

Keyword cost = vanilla stats at that mana minus the card's stats, for cards whose *only* modifier is that keyword:

| Keyword (text from `localization/locales/en/modifiers.json`) | Cards | Mean stats given up | Example |
|---|---|---|---|
| Provoke ("must attack this first") | 5 | 2.1 | Primus Shieldmaster 3/6 for 4 (`core/neutral.coffee` 629) |
| Flying ("move anywhere") | 6 | 2.5 | Flame Wing 5/4 for 4 (1323) |
| Celerity ("activate twice") | 3 | 2.3 | small sample |
| Frenzy (hits all adjacent) | 2 | 3.3 | Serpenti 7/4 for 6 (811) |
| Blast (line) | 1 | 3.0 | Pyromancer 2/1 for 2 (`core/faction3.coffee` 277) |
| Ranged ("attack from anywhere") | 5 | 4.2 | Fire Spitter 3/2 for 4 (`core/neutral.coffee` 112) |
| Forcefield (cancel first damage each turn) | 2 | 4.8 | `monthly/m6_forcefield.coffee` 42, 75 |
| Rush ("move and attack immediately") | 2 | 8.0 | Saberspine Tiger 3/2 for 4 (`core/neutral.coffee` 1589) |
| Backstab, Airdrop | 2 each | ≈0–1 | |

- Acting at once (Rush) is the most expensive keyword. In hello1 terms that is a robot that attacks without a telegraph. Never allow it **(inference)**.
- Challenges (`app/sdk/challenges/*`) are hand-built one-turn puzzles with no budget formula.

## 2. Wesnoth: unit cost and campaign difficulty

**Unit cost** (parsed `data/core/units/*/*.cfg`, base unit only):

| Level | Units | Median cost |
|---|---|---|
| 0 | 21 | 10 |
| 1 | 78 | 16 |
| 2 | 109 | 29 |
| 3 | 94 | 56 |

Level-1 least squares: cost ≈ 1.5 + 0.07·HP + 1.22·moves + 0.28·(best damage × strikes) + 0.73·(has both ranges). R² = 0.36, mean error 2.1 gold.

| Unit | Cost | HP | Moves | Best attack | File |
|---|---|---|---|---|---|
| Grunt | 12 | 38 | 5 | 9×2 | `orcs/Grunt.cfg` |
| Spearman | 14 | 36 | 5 | 7×3 | `humans/Loyalist_Spearman.cfg` |
| Cavalryman | 17 | 34 | 8 | 6×3 | `humans/Loyalist_Cavalryman.cfg` |
| Heavy Infantryman | 19 | 38 | 4 | 11×2 | `humans/Loyalist_Heavy_Infantryman.cfg` |
| Mage | 20 | 24 | 5 | 7×3 magic | `humans/Mage.cfg` |
| Gryphon Rider | 23 | 34 | 8 (fly) | 12×2 | `gryphons/Gryphon_Rider.cfg` |

- A move point is worth about 1.2 gold, or 7.5% of a median unit. A point of HP is worth almost nothing on the margin.

**Campaign difficulty.** The macros `{GOLD e n h}` and `{INCOME e n h}` (`data/core/macros/utils.cfg` lines 96–111) and `ON_DIFFICULTY` (lines 46–55) set side values per difficulty.

| HttT 03 Blackwater Port | Easy | Normal | Hard | Nightmare |
|---|---|---|---|---|
| Enemy starting gold (4 sides) | 135 | 250 | 365 | 475 |
| Enemy income per turn | 8 | 24 | 40 | 56 |
| Player gold / income | 75 / +0 | same | same | same |

Sources: `03_Blackwater_Port.cfg` lines 46–47, 68–69, 90–91, 173–174; `unified_characters.cfg` lines 11–16; `data/game_config.cfg` line 16 (`base_income=2`).

I also parsed 66 scenarios that use 3-value macros on one human side and AI sides:

| Ratio (median) | Value |
|---|---|
| Enemy ÷ player starting gold, Easy / Normal / Hard | 0.65 / 1.07 / 1.58 |
| AI gold, Hard ÷ Easy | 1.67 |
| Player gold, Hard ÷ Easy | 0.75 (lower on Hard in 49 of 66, equal in 14) |

- The player's growth is never measured. Veterans come back through the recall list (`recall_cost=20`, `game_config.cfg` line 21), and gold carries over at 40% (`03_Blackwater_Port.cfg` line 368). The gold ratios above leave that army out, so they understate the player.
- Designers plan around an *expected* player state instead: "Konrad will probably win around turn 22, with 0 gold in the bank, for ~60 carryover gold" (`02_Flight_of_the_Elves.cfg` line 8).
- I found no Wesnoth source saying the AI gets more because it plays worse. That is my **(inference)**, supported by ITB's note that "Enemies never plan future moves" (see `enemy-ai.md`).

## 3. Into the Breach: spawns by progress

| Setting (Normal unless noted) | Value | Source |
|---|---|---|
| Starting enemies by sector 1–4 | 3, 3, 4, 4 (Easy 2, 2, 3, 3; Unfair 5) | `missions/missions.lua` 1001–1003 |
| Spawns per turn | 2, 3 alternating (Easy 2, 1; Unfair 5, 5) | 998–1000 |
| Most enemies on the board | 6 | 86, 962 |
| No spawns on the final turn | – | 940, 968–970 |
| +1 spawn if ≤2 enemies remain (not on Easy) | – | 957–960 |
| Weak share of spawns | 4/5, then 3/5 in sector 4 | `spawner.lua` 47–69 |
| Alpha share, sector 1–4 (Easy / Normal / Hard) | 0,0,1,2 / 0,1,2,4 / 1,2,3,5 out of 5 | `spawner.lua` 16–101 |
| Most Alphas alive at once (Normal) | 0, 2, 3, 5 | `spawner.lua` 50–68 |
| Enemy types available | 3 + sector | `game.lua` line 102 |

- The squad is always 3 mechs. At the cap, that is 2 enemies per mech.
- Difficulty first raises enemy *quality* (the Alpha share), and count only a little. Comments show both were cut during tuning ("WAS 3", "WAS 5"; `spawner.lua` 60–68).
- Nothing reads the squad's power. Progress (the sector) drives everything.
- Rough whole-mission tally **(inference)**: 3 starting + about 7 spawned Vek, at roughly 6 of my points each, against about 24 for the squad (formula below). That is about 2.5× over a survive-4-turns mission and about 1.5× on the board at the peak.

## 4. Tabletop encounter budgets

**D&D 5e, 2014 Basic Rules** ([source](https://www.dndbeyond.com/sources/basic-rules/building-combat-encounters)):

| Monsters | 1 | 2 | 3–6 | 7–10 | 11–14 | 15+ |
|---|---|---|---|---|---|---|
| XP multiplier | ×1 | ×1.5 | ×2 | ×2.5 | ×3 | ×4 |

Per-character thresholds at level 1 are 25/50/75/100 (Easy/Medium/Hard/Deadly). Parties under 3 use the next-higher multiplier. Parties of 6 or more use the next-lower one.

**D&D 2024 free rules** ([DM's Toolbox](https://www.dndbeyond.com/sources/dnd/free-rules/dms-toolbox)): the budget per character at level 1 is 50/75/100 (Low/Moderate/High), multiplied by the number of characters. It has no count multiplier. Guidance: "more than two creatures per character → include fragile creatures", "especially important for characters of level 1 or 2", and "a single monster generally presents a low-difficulty challenge for a party of four".

**PF2e GM Core** ([AoN](https://2e.aonprd.com/Rules.aspx?ID=2716)):

| Threat | Budget (4 PCs) | Per extra or missing PC | | Creature level vs party | XP |
|---|---|---|---|---|---|
| Low | 60 | 20 | | −4 / −2 / −1 | 10 / 20 / 30 |
| Moderate | 80 | 20 | | equal | 40 |
| Severe | 120 | 30 | | +1 / +2 | 60 / 80 |
| Extreme | 160 | 40 | | +3 / +4 | 120 / 160 |

- Cost doubles every 2 levels. A single +4 creature fills the whole Extreme budget.
- Count is priced in: four level −2 lackeys cost the same as one level +2 boss.

## 5. Dynamic difficulty and growing parties

| System | What it scales on | Upgrades still feel good? |
|---|---|---|
| ITB | Progress (sector) plus a small count rubber band | Yes: enemies ignore squad power |
| Wesnoth campaign | Difficulty setting; the player's army isn't measured | Yes |
| D&D / PF2e | Party level × party size (full scaling) | Mostly: items and tactics aren't measured |
| Duelyst AI | Wins: `min(1, wins/10)` (see `enemy-ai.md`) | Yes, it tracks skill, not deck |
| L4D Director | Player "intensity"; changes frequency, not amplitude ([transcript](https://www.readkong.com/page/the-ai-systems-of-left-4-dead-michael-booth-valve-9664541)) | Yes |

- **Growing parties.** Wesnoth lets the army grow and leaves enemy gold fixed. PF2e and D&D 2024 scale linearly per character. D&D 2014 also makes small parties face a harsher count multiplier.
- **Full scaling is a rubber band.** If enemies match every upgrade, the ratio never moves and upgrades feel pointless. None of the first-party systems I checked scales on the player's *gear*. They scale on progress, difficulty, level or skill **(inference from the table)**.

## 6. Cross-source principles (inference)

1. Count is its own axis. Cap enemies on the board relative to team units, and price extra bodies above their stats.
2. Use fragile bodies to add count (D&D 2024; ITB's 1-HP Bots in `hp-scale.md`).
3. Scale on an expected curve by progress. Let the player's real deviation leak through only partly.
4. Changes in party size pass straight into the budget (PF2e), because each body is a whole action per turn.
5. Formulas are a first guess. Wesnoth's R² of 0.36 and Duelyst's noisy keyword costs show that playtesting set the final prices.
6. The enemy may have more points because it plays worse. It never plans ahead, it telegraphs, and the player moves first.

## 7. Recommended system for hello1 (inference)

Everything in this section is **(inference)**.

### 7.1 Unit points

**P = HP + 2·Dmg + Move + Reach + Displace + Ability − Weakness**

| Term | Rule | Change from draft and why |
|---|---|---|
| HP | 1 per HP. **The human counts ⌈HP/2⌉.** | The human's HP is a lose-condition buffer and the AI focuses him (+8). Duelyst's general has 25 HP but deals only 2. |
| Dmg | 2 per point of main attack | Kept. Duelyst weights attack and HP about equally, but in hello1 1 damage often decides hits-to-kill. |
| Move | move − 3, from −1 to +2 | Kept. Wesnoth prices each move point at ~7.5% of a unit. |
| Reach | melee 0, line or charge +1, ranged +2 | **Swapped.** Duelyst Ranged costs ~4 and Blast ~3. A line attack is blocked by the first unit. |
| Displace | +1 per tile pushed or pulled | Kept |
| Ability | cancel an attack 3, shield 3, fly 2, wall 2, counter 2, taunt 2, unpushable 1, bump-immune 1, each extra telegraph per turn +3 | Matches the Duelyst costs. A robot that attacks without a telegraph is banned (Rush costs 8). |
| Weakness | −1 each | Kept |
| Parts | price the change with the same formula; halve it if cooldown ≥ 2 or it needs a second part | Cooldown limits how often a part fires |

Sanity check (stats from `animal-roster.md`):

| Unit | Calculation | P | Draft |
|---|---|---|---|
| Human (6 HP, crowbar 1 + push 1, flare ranged, hack) | 3+2+0+2+1+3 | **11** | 13 |
| Bear | 4+4+0+0+1+1 | **10** | 9 |
| Boar (charge, bump-immune, charge also moves it +1) | 3+2+0+1+1+1+1 | **9** | 9 |
| Ram | 3+2+0+0+2+1 (climb) | **8** | – |
| Eagle | 3+2+0+0+1+2 | **8** | – |
| Skunk (cancel on 2 tiles) | 3+0+0+0+0+4 | **7** | – |
| Porcupine | 3+2+0+0+0+2 | **7** | – |
| Beaver | 3+0+0+0+0+2+1 (swim) | **6** | – |
| Dog (drag, no damage) | 3+0+1+0+1 | **5** | 7 |
| Scout drone, Light (1 HP, move 4, fly) | 1+2+1+0+0+2−1 | **5** | ~6 |
| Taser, Electric (2 HP, ranged 1) | 2+2+0+2−1 | **5** | ~6 |
| Fuel Tank (3 HP, 2 dmg) | 3+4−1 | **6** | ~6 |
| Heavy Walker (4 HP, 2 dmg, move 2, push 1) | 4+4−1+1−1 | **7** | ~6 |
| Crusher, Heavy (4 HP, 3 dmg, move 2) | 4+6−1−1 | **8** | – |
| Elite (+2 HP, +1 dmg) | base + 4 | **+4** | – |
| Boss (7 HP, 3 dmg, ranged, 2 telegraphs, unpushable) | 7+6+2+3+1 | **19** | – |

The ranking looks right. The Bear and Boar are the sturdy fighters. The Dog scores lowest, but its value comes from combos (dragging robots into attacks), which a points formula can't see. Keep the Dog as the starter anyway, and let the solver measure what it is actually worth.

### 7.2 Count multiplier

M depends on robots *active this turn* minus team units:

| Robots − team units | ≤ −1 | 0 | +1 | +2 | ≥ +3 |
|---|---|---|---|---|---|
| M | 0.8 | 1.0 | 1.25 | 1.5 | 2.0 (avoid) |

- Team of 2 units (the start): move one step right, as D&D does for parties under 3.
- Hard caps: robots on the board ≤ team units + 2, and ≤ 2 per team unit. Any robot past team units + 1 must be Light (1–2 HP).
- A lone boss gets 0.8. Give it escorts or a second telegraph so it still pressures the team.

### 7.3 Budget rule: expected curve plus partial scaling

1. **T** = the team's actual P total (units + parts).
2. **T_ref** = the expected T for this stage, *recomputed with the team's actual unit count*.
3. **T_b = T_ref + 0.5·(T − T_ref)**, in both directions.
4. **Battle total** (initial robots + drop-ins): ΣP ≤ R·T_b.
5. **Per-turn peak:** ΣP(active robots) × M ≤ k·T_b. A drop-in lands only if this check still passes, like ITB's `MaxEnemy − enemies` (`missions.lua` 962).

| Stage | Expected team | T_ref | R (survive 4 turns) | k |
|---|---|---|---|---|
| Early (battles 1–3) | Human + Dog, 0 parts | 16 | 1.4 | 0.9 |
| Mid (4–6) | + Bear, 3 parts (~2 each) | 32 | 1.6 | 1.1 |
| Late (7–9) | + Boar, 6 parts | 47 | 1.8 | 1.3 |
| Boss | as late | 47 | 1.8 | 1.4 |

If the objective is "destroy all robots" instead, use 0.8·R.

**Why not A or B?**
- **A (enemy = team × ratio)** scales fully. A +8 upgrade brings about +11 enemy points, so the ratio never moves and upgrades feel pointless.
- **B (base + 50% of growth since the run start)** anchors on the starting team. It therefore counts *expected* growth twice. It also treats a new teammate like a part, but a body is a whole action per turn.
- **This rule** follows ITB and Wesnoth's fixed curves, uses PF2e's per-unit adjustment for team size, and gives only half the rubber band on parts.

### 7.4 Worked examples

**Start: battle 1, human + Dog.** T = 16, N = 2, T_b = 16. Total ≤ 22. Peak ≤ 14.4.
- Turn 1: Fuel Tank 6 + Scout 5 = 11. Two robots against two units, moved one step for the small team: ×1.25 → 13.75. ✓
- A Taser drop-in (5) while both are alive: 16 × 1.5 = 24. ✗ It waits until one robot dies (5 × 1.25 = 6.25. ✓).
- Budget left after that: 6. One more Tank drop-in later. That is 4 robots in all, about ITB Easy's 2 + 2.

**Mid: battle 5, human + Dog + Bear + 3 parts.** T = 32 = T_ref. Total ≤ 51. Peak ≤ 35.
- Turn 1: Heavy Walker 7 + Tank 6 + Taser 5 + Scout 5 = 23, ×1.25 = 28.75. ✓
- Drop-ins: Elite Tank 10, Flamer 5, Scout 5, Crusher 8. Total 51.
- **Strong player** (T = 40): T_b = 36. Total ≤ 58, which adds one robot. Under A it would be +11. With no scaling it would be +0.
- **Bear died** (N = 2): T = 22. T_ref is recomputed for 2 units: 16 + 6 = 22. Total ≤ 35, peak ≤ 24, and the small-team shift applies.

**Late: battle 8, human + 3 animals + 6 parts.** T = 47. Total ≤ 85. Peak ≤ 61.
- Turn 1: Crusher 8 + Elite Tank 10 + Walker 7 + Taser 5 + Scout 5 = 35, ×1.25 = 43.75. ✓
- About 50 points of drop-ins over turns 2–3, for example 2 elites + 4 normal robots, each gated by the peak check.
- With only 2 animals: T_ref = 38. Total ≤ 68.

**Boss, late team.** Boss 19 + 2 Scouts on turn 1 = 29, ×0.8 = 23 (well under the peak). Spend the rest as escorts that drop in, so the boss never stands alone.

### 7.5 Telegraph and first-move advantage

The telegraph is priced into R and k, not into unit points. The player sees every attack and moves first. A threat usually takes one action to dodge, block or push aside, while killing the robot takes several. So the team only needs to keep up with *telegraphs per turn*, not with robot HP. Calibration: ITB runs about 1.5× at the peak and 2.5× over a whole battle, with mechs that come back. hello1 has permanent death and the "slightly easy" rule, so start lower (1.4–1.8) and raise it from data.

## 8. Validation plan (inference)

**One-turn solver.** The game is deterministic, so enumerate every unit order (N!) × each unit's moves (~20) × actions (~8). For 3 units: 6 × 160³ ≈ 25M states before deduplication, which is fine in compiled code. For 4 units, deduplicate states and keep the top 1,000 per unit step (beam search). Score each end state: human damage, then animal deaths, then habitat damage, then animal damage, then robots killed.
- Per turn, report: zero-damage lines exist (yes/no), how many lines are optimal, the gap between the best and second-best lines, and whether a forced loss exists.
- **Battle sim:** run the deterministic AI against (a) the solver and (b) a "human-like" bot that picks randomly among the top 20% of lines. Use 200 seeds per stage.

**Targets:**

| Check | Early | Mid | Late | Boss |
|---|---|---|---|---|
| Solver: turns with no zero-damage line | 0% | ≤ 5% | ≤ 15% | ≤ 25% |
| Solver: forced unit loss | 0% | 0% | 0% | ≤ 2% |
| Human-like bot: battle lost or animal lost | ≤ 5% | ≤ 10% | ≤ 15% | ≤ 25% |
| Playtesters: full-run win rate | ~50% for an experienced player | | | |

**Log every battle:** T, T_ref, T_b, enemy P spent (initial and drop-ins), the per-turn peak ×M, robots active against N, damage by unit, deaths, habitat losses, turns taken, undo count, seconds per turn, parts and combos fired, and the gap between the player's score and the solver's best each turn ("regret").

**Fit the weights from data:** model loss chance as a function of ΣP/T_b, (robots − N) and the stage. Retune R, k, M and the term weights. Watch three outliers:
- Units whose real value differs most from their P. The Dog is the likeliest.
- Parts whose high pick rate goes with a high win rate (`balancing-tactics-games.md`).
- Battles that the solver clears in one turn.

## Unverified

- **SRD 5.1 itself.** I cited the 2014 Basic Rules and the 2024 free rules on D&D Beyond (both official). I didn't open the SRD 5.1 PDF, so I can't confirm that it contains the encounter multiplier table.
- **Booth's L4D slides.** The Valve and Steam CDN PDF was blocked from here, so the quotes come from a readkong transcript.
- **Slay the Spire Ascension and Hades Heat.** I found no first-party design source. Both are opt-in difficulty layers, which is how they avoid the "upgrades feel pointless" problem, but that is unsourced here.
- **Wesnoth parse.** It covers only scenarios with 3-value macros (66 of them) and leaves out recall lists and carryover.
- **Duelyst keyword costs** come from 2–6 cards each. Rarity and faction effects aren't controlled for.
- **ITB.** Whether "sector" means islands completed is my reading of `GetSector()`. The 2.5× whole-mission ratio uses my point formula on ITB units, not ITB's own numbers.
- **All hello1 stats** (human attack, Dog damage, robot moves) are my reading of the drafts and still need confirming.
