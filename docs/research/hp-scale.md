# HP scale for the leader, animals and robots

Research notes for hello1. Researched 2026-11-06. Claims are cited with a URL or a local path. Anything marked **(inference)** is my own reasoning and not sourced. This file extends `balancing-tactics-games.md` (which already notes Duelyst's 25 HP generals and a rough "1–4 HP robots" target) and does not repeat it. Local repo paths are relative to `game-projects/`.

ITB script paths below are from a third-party copy of the game's Lua scripts: https://github.com/2147211690/LearnBreach (`scripts/...`). `enemy-ai.md` explains why that copy looks genuine. Line numbers refer to that copy.

## 1. Summary

1. **ITB keeps everything between 1 and 7 HP.** Mechs have 2–3 HP. Normal Vek have 1–4 HP (median 3), Alpha Vek 3–5 (median 4.5), and bosses 5–7 (`scripts/pawns.lua`; `scripts/missions/bosses/*.lua`).
2. **ITB damage is 1–3 per hit, plus 1 for a bump.** Starting mech weapons deal 1–3. Normal Vek attacks deal 1 (the Leaper deals 3). Alpha attacks deal 2–3 (the Alpha Leaper deals 5) (`scripts/weapons_*.lua`). "Pushing units into things deals 1 additional damage to both!" (`scripts/text.lua` line 166).
3. **ITB's key units are fragile.** A mech survives 2–3 normal Vek hits but only one Alpha hit. A normal Vek dies to 1–2 player actions. An Alpha needs 2–3. A boss needs 3–4.
4. **ITB has no "leader" unit.** The run-ending pool is the power grid, not a mech. Mechs are repaired after every battle (`scripts/text.lua` line 142). hello1's animals die permanently, so they need more margin than ITB mechs **(inference)**.
5. **Leader HP versus ordinary units varies a lot.** Wesnoth's multiplayer leaders are ordinary level-2 units, about 1.2× a level-1 recruit's HP. Duelyst's general has 25 HP, about 8× a median minion (3 HP). ITB's grid is 7 points against mech HP of 2–3 (sections 3–5).
6. **Hits-to-kill clusters at 1–3 for ordinary units in every game I checked.** The exceptions are leaders (Duelyst ~8 hits) and boss-like enemies (ITB 3–4, Slay the Spire ~7) (section 7).
7. **Recommendation (inference):** human 6, animals 3 (one sturdy animal at 4), robots 1–4 by nature, elite robots +2, bosses 6–7. Damage: 1 basic, 2 strong, 3 combo, bump 1. No single robot attack deals more than 2 to an animal in early battles (section 8).

## 2. Into the Breach

### Unit HP

| Unit | HP | Source |
|---|---|---|
| Combat Mech (Prime) | 3 | `pawns.lua` line 11 |
| Cannon Mech (Brute) | 3 | `pawns.lua` line 28 |
| Artillery Mech (Ranged) | 2 | `pawns.lua` line 43 |
| Other mechs | 2–3 (Ranged and flying mechs usually 2) | `pawns.lua` lines 60–443 |
| Pilot skill "Health Bonus" | +2 mech HP | `text.lua` lines 528–530 |
| Pilot skill "Rockman" | +3 HP | `text_tooltips.lua` line 131 |
| Ralph (starting pilot) | `HealthBonus = {2,0}` | `pilots.lua` line 82 |

Vek and Bots, normal and Alpha:

| Enemy | Normal HP | Alpha HP | Source (`pawns.lua`) |
|---|---|---|---|
| Leaper | 1 | 3 | lines 725, 740 |
| Scarab | 2 | 4 | 789, 807 |
| Digger | 2 | 4 | 895, 910 |
| Hornet | 2 | 4 | 927, 944 |
| Spider | 2 | 4 | 1024, 1040 |
| Scorpion | 3 | 5 | 659, 674 |
| Firefly | 3 | 5 | 690, 707 |
| Crab | 3 | 5 | 826, 843 |
| Centipede | 3 | 5 | 861, 878 |
| Blobber | 3 | 4 | 575, 590 |
| Burrower | 3 | 5 | 1111, 1124 |
| Beetle | 4 | 5 | 757, 772 |
| Psion (Jelly) | 2 | – | 980 |
| Snow Bots (all four) | 1 | 1 | 1179–1294 |

Alpha is usually normal +2 HP. The Bot entries carry comments with older values (`Health = 1, --3`), so the shipped 1 HP was a deliberate cut.

Bosses:

| Boss | HP | Attack damage | Source |
|---|---|---|---|
| Scorpion Leader | 7 | 2 | `missions/bosses/scorpion.lua` lines 12, 28 |
| Hornet Leader | 6 | 2 | `hornet.lua` lines 12, 30 |
| Beetle Leader | 6 | 3 | `beetle.lua` lines 12, 29 |
| Spider Leader | 6 | – | `spider.lua` line 51 |
| Psion Abomination | 5 | – | `psion.lua` line 15 |
| Bot Leader | 5 (6 in a second form) | 2 / 4 | `bot.lua` lines 44, 67, 73, 79 |

### Damage

| Attack | Damage | Source |
|---|---|---|
| Titan Fist (Combat Mech) | 2 (upgrade +2) | `weapons_prime.lua` lines 43, 52 |
| Taurus Cannon | 1 | `weapons_brute.lua` line 47 |
| Artemis Artillery | 1 (upgrades +1, +1) | `weapons_ranged.lua` lines 59, 66 |
| Burst Beam | 3, falling to a 1 minimum | `weapons_prime.lua` lines 243–245 |
| Rocket, Lightning, Flamethrower | 2 | `weapons_ranged.lua` 249; `weapons_prime.lua` 138, 661 |
| Normal Vek attacks | 1 (Leaper 3) | `weapons_enemy.lua` lines 4–565 |
| Alpha Vek attacks | 2–3 (Alpha Leaper 5) | `weapons_enemy.lua` lines 21–611 |
| Push into an object | +1 to both | `text.lua` lines 166–169 |
| Fire | 1 per turn | `text_tooltips.lua` line 84 |
| Armor | −1 weapon damage; push, block and fire unaffected | `text_tooltips.lua` line 66 |
| A.C.I.D. | ×2 weapon damage; push, block and fire unaffected | `text_tooltips.lua` line 89 |

Note that armor and A.C.I.D. touch only weapon damage. Bump damage is always a flat 1. That keeps the bump easy to count.

### Hits-to-kill in ITB

| Attacker → target | Hits |
|---|---|
| Mech weapon (1–2) + bump (1) → normal Vek (3) | 1–2 |
| Same → Alpha Vek (4–5) | 2–3 |
| Same → boss (5–7) | 3–4 |
| Normal Vek (1) → mech (2–3) | 2–3 |
| Alpha Vek (2–3) → mech (2–3) | 1–2 |
| Leaper (3) or Alpha Leaper (5) → mech | 1 |

Three mechs get three actions per turn. A normal Vek costs one or two of them. So a turn removes about one to three Vek **(inference)**.

## 3. Duelyst

All numbers come from `duelyst/app/sdk/cards/factory/core/*.coffee`. I parsed every card block that sets `atk`, `maxHP` and `manaCost` (207 minions and 18 generals).

| Unit | Attack | HP | Source |
|---|---|---|---|
| Every general (18) | 2 | 25 | e.g. `faction1.coffee` lines 118–119 |

Minions by mana cost:

| Mana | Cards | Median attack | Median HP | HP range |
|---|---|---|---|---|
| 1 | 24 | 1.5 | 1 | 1–4 |
| 2 | 45 | 2 | 3 | 1–6 |
| 3 | 46 | 2 | 3 | 1–8 |
| 4 | 38 | 4 | 4 | 1–8 |
| 5 | 23 | 4 | 5 | 3–10 |
| 6 | 16 | 5.5 | 5.5 | 1–8 |
| 7 | 13 | 5 | 10 | 5–15 |

Across all 207 minions, the median is 3 attack and 3 HP.

- A minion against a same-cost minion dies in about 1 hit. Attack roughly equals HP at every cost.
- The general takes 25 ÷ 3 ≈ 8 median minion hits.
- The general only deals 2 per hit, so it is a poor attacker. Its large HP is a lose-condition buffer, not combat power **(inference)**.
- Duelyst's AI treats low-HP generals as "evasive" and keeps them away from combat (`enemy-ai.md`, citing `scoring/position/...`).

## 4. Wesnoth

I parsed `wesnoth/data/core/units/*/*.cfg` for `hitpoints`, `level`, `damage` and `number` (strikes).

| Level | Units | Median HP | Median best single hit | HP ÷ level-1 hit |
|---|---|---|---|---|
| 0 | 21 | 20 | 5 | 2.9 |
| 1 | 79 | 33 | 7 | 4.7 |
| 2 | 109 | 47 | 9 | 6.7 |
| 3 | 95 | 62 | 11 | 8.9 |

Example units:

| Unit | HP | Main attack | Source |
|---|---|---|---|
| Spearman (L1) | 36 | 7 × 3 | `Loyalist_Spearman.cfg` lines 8, 87–88 |
| Grunt (L1) | 38 | 9 × 2 | `orcs/Grunt.cfg` lines 8, 92–93 |
| Lieutenant (L2 leader) | 40 | 8 × 3 | `Loyalist_Lieutenant.cfg` lines 9, 29–30 |

- A Spearman needs 5 Grunt hits (38 ÷ 9 rounds up to 5). Hits land about 60% of the time on flat ground (`smallfoot` flat=60, `data/core/units.cfg` line 438). So a kill takes two or three rounds of combat.
- Multiplayer leaders are drawn from normal level-2 units: "leader=Lieutenant,Swordsman,Pikeman,..." (`data/multiplayer/factions/loyalists-default.cfg` line 7). The Lieutenant's 40 HP is only 1.2× the median level-1 unit's HP. Wesnoth gives its leader no special HP. Leaders are protected by position and the player's choices.
- Wesnoth needs large numbers because of RNG and resistance percentages. With no randomness, hello1 does not need that granularity **(inference)**.

## 5. Slay the Spire (wiki pages, used as a check only)

| Value | Number | Source |
|---|---|---|
| Ironclad max HP | 80 | [wiki](https://slaythespire.wiki.gg/wiki/Ironclad) |
| Strike | 6 damage | [wiki](https://slaythespire.wiki.gg/wiki/Strike) |
| Jaw Worm HP | 40–44 | [wiki](https://slaythespire.wiki.gg/wiki/Jaw_Worm) |
| Jaw Worm Chomp / Thrash | 11 / 7 | [wiki](https://slaythespire.wiki.gg/wiki/Jaw_Worm) |

- A Jaw Worm takes about 7 Strikes to kill. That is a whole fight over several turns.
- The Ironclad takes 7–8 Chomps to kill. HP carries over between fights, so the player's HP is a run-long pool, like ITB's grid.
- StS is a one-on-one card fight, not a grid game. Its larger numbers serve that pacing and don't map onto hello1 **(inference)**.

## 6. Leader and key-unit HP

| Game | Key unit | Key unit HP | Ordinary unit HP | Ratio |
|---|---|---|---|---|
| ITB | Power grid (run pool) | 7 (unverified) | Mech 2–3 | ~2.5–3× |
| Duelyst | General | 25 | Minion median 3 | ~8× |
| Wesnoth | MP leader (L2) | ~40–47 | L1 median 33 | ~1.2–1.4× |
| StS | Player | 80 | Normal enemy ~40–50 | ~2× |

The pattern: when losing the key unit ends the game *and* the enemy focuses it, its HP sits at 2–8× ordinary units. The hello1 human is focused (weight +8 against +5 for an animal) and ends the run, so it belongs in that band. A ratio of about 2× fits a small board where every hit is visible **(inference)**.

## 7. Cross-game hits-to-kill

| Matchup | ITB | Duelyst | Wesnoth (if all hit) | StS |
|---|---|---|---|---|
| Player → ordinary enemy | 1–2 | ~1 | ~5 | ~7 |
| Player → elite / boss | 2–4 | ~2–3 (7-mana: 10 HP ÷ 4) | ~7–9 | – |
| Enemy → player's ordinary unit | 2–3 (1 vs Alpha) | ~1 | ~4–5 | – |
| Enemy → leader / run pool | – | ~8 | ~4–6 | 7–8 |

The deterministic grid game (ITB) sits at the low end: 1–3 hits for everything except bosses. The RNG games (Wesnoth) and duel games (StS) pad HP to stretch fights. The small range is what lets an ITB player count lethal at a glance **(inference)**.

## 8. Recommended numbers for hello1 (inference)

All of this section is **(inference)** from the tables above.

**Damage scale**

| Source | Damage |
|---|---|
| Basic attack (animal or robot) | 1 |
| Strong attack / heavy robot | 2 |
| Combo payoff | 3 (rarely 4) |
| Bump when a push is blocked | 1 to both (ITB rule) |
| Elite robot attack | 2–3 |

**HP**

| Unit | HP | Hits to kill | Why |
|---|---|---|---|
| Human (leader) | 6 | 6 basic, 3 strong | 2× an animal. Two robots hitting for 2 each leave it at 2 HP. |
| Animal | 3 | 3 basic, 2 strong | Matches ITB mechs. Permanent death means robots should not one-shot an animal early. |
| Sturdy animal (trait) | 4 | 4 basic, 2 strong | A sidegrade, like Wesnoth's traits. |
| Calmed wild animal | 2–3 | 2–3 | A temporary helper, so it can be more fragile. |
| Light robot | 1–2 | 1 action | Like ITB Bots (1 HP) and Leapers. |
| Fuel / Electric robot | 2–3 | 1–2 actions | Like normal Vek (median 3). |
| Heavy robot | 4 | 2–3 actions | Like the Beetle (4). Pairs well with resistance. |
| Elite robot | normal +2 | 2–3 actions | The ITB Alpha pattern. |
| Boss robot | 6–7 | 3–4 actions | Like ITB bosses (5–7). |

**Rules of thumb**

- Keep every value in 1–6, plus bosses at 6–7. Every number then fits on one die face, and lethal checks are single-digit sums.
- Keep "max single hit ≤ animal HP − 1" in early battles. With animals at 3, that caps a robot hit at 2. ITB's Leaper one-shots mechs, but mechs come back. Animals don't.
- The human at 6 means three robots focusing it for 2 each would kill it in one turn. That is fine only if the player can always block or push at least one attacker. Map generation should check this, as `balancing-tactics-games.md` suggests with a solver. If playtests show too many sudden deaths, try human 7 (ITB's grid size) before raising robot HP.
- Make the human's HP carry across battles if you want it to act as the run pool, like ITB's grid and StS's HP. Otherwise heal it between battles. That choice changes how 6 HP feels more than the number itself does.
- Your proposal (human 6, animals 3–4, robots 2–4) fits the references. The only changes I'd suggest: let Light robots go down to 1 HP, and add elite (+2) and boss (6–7) tiers instead of raising base HP.

## 9. Unverified

- **ITB grid power max of 7.** `scripts/game.lua` line 252 has `getCityPower(sector) return 7`, but the function's meaning is not clear from the code. Fan wikis say 7. I did not fetch them; `balancing-tactics-games.md` notes they refuse automated fetches.
- **ITB building HP.** `getBuildingHealth` (`game.lua` lines 386–391) returns tables like `{50,40,10,0}`. These look like percentages of 1-, 2-, 3- and 4-HP buildings by island, but I could not confirm that.
- **Mech HP upgrades from reactor cores.** `text.lua` line 572 has the label "+2 Health", but I did not find the rule that applies it.
- **Whether the LearnBreach copy matches the shipped game exactly.** The Bot HP comments suggest a development snapshot or an Advanced Edition build. The mech and Vek values match what I remember of the game, but I did not check them against a retail install.
- **Bump damage being outside Lua.** The rule appears only in tutorial text (`text.lua` line 166) and the Force Amp passive (`text_weapons.lua` line 234). The engine code is not in Lua.
- **Fights in Tight Spaces.** I found no first-party or file-quoting source for its HP values, so it has no section.
- **Slay the Spire numbers** come from the community wiki, not the game files.

## 10. Sources

- ITB scripts (third-party copy): https://github.com/2147211690/LearnBreach — `scripts/pawns.lua`, `pilots.lua`, `game.lua`, `text.lua`, `text_tooltips.lua`, `text_weapons.lua`, `weapons_prime.lua`, `weapons_brute.lua`, `weapons_ranged.lua`, `weapons_enemy.lua`, `missions/bosses/*.lua`
- Slay the Spire wiki: https://slaythespire.wiki.gg/wiki/Ironclad, https://slaythespire.wiki.gg/wiki/Strike, https://slaythespire.wiki.gg/wiki/Jaw_Worm
- Local: `duelyst/app/sdk/cards/factory/core/*.coffee`
- Local: `wesnoth/data/core/units/*/*.cfg`, `wesnoth/data/core/units.cfg`, `wesnoth/data/multiplayer/factions/loyalists-default.cfg`
