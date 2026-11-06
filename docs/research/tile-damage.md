# Tile and Hazard Damage

How tactics games set damage from tiles, terrain and effects against unit HP and normal hits. The goal is tile-damage numbers for hello1.

This extends `hp-scale.md` (HP and weapon numbers), `balancing-tactics-games.md` (resistance tables, solver checks) and `enemy-ai.md` (ITB AI hazard penalties, line 40). It does not repeat them.

ITB paths are from the third-party script copy at https://github.com/2147211690/LearnBreach (`scripts/...`). `enemy-ai.md` explains why that copy looks genuine.

## Key findings

1. **Damage over time is 1 per turn in ITB and Duelyst.** ITB: "Units on Fire take 1 damage at the start of every turn" (`text_tooltips.lua` line 84). Duelyst Shadow Creep deals 1 by default (`modifiers/modifierStackingShadows.coffee` line 25). Against a typical 3-HP unit that is about a third of its HP per turn.
2. **One tick of a hazard is never more than one normal hit.** ITB fire (1) equals a normal Vek attack (1). Wesnoth poison (8 per turn, `data/game_config.cfg` line 19) is about one level-1 hit (7–9, `hp-scale.md`). Duelyst creep (1) is below the median minion attack (3).
3. **ITB fire weapons deal 0 direct damage.** Vulcan Artillery and Fire Beam have `Damage = 0` (`weapons_ranged.lua` line 312; `weapons_science.lua` line 560). Fire is the whole payoff, spread over turns. The Flame Thrower adds +2 only to units *already* on fire (`weapons_prime.lua` lines 661–662, 687–698). That is a two-step combo.
4. **Instant kills are terrain, not damage.** ITB water, chasm, mines, air strikes, lightning and falling rocks "kill any unit" (`text_tooltips.lua` lines 20, 25, 37–46). They ignore HP entirely. Each is shown ahead of time, with a "Deadly Threat" tooltip (line 106).
5. **Instant kills are filtered by unit type, and the filter favours the player.** "Most non-flying enemies die in Water" (line 25). Every player mech is `Massive` and survives water but can't shoot (`pawns.lua` line 19; tooltip line 80). Bosses are "Immune to water" (line 108).
6. **Flat hazard damage ignores multipliers.** ITB armor (−1) and A.C.I.D. (×2) touch only weapon damage. "All other damage (Push, Fire, Blocking, etc.) is unaffected" (`text.lua` lines 95, 107). This keeps hazard sums easy to count.
7. **Some hazards hit only the enemy.** Duelyst Shadow Creep hits only enemies (`modifierStackingShadows.coffee` lines 94–95). ITB Storm Generator smoke "damages enemy units" (`text_tooltips.lua` line 7). Others hit everyone: ITB fire, bump, Electric Whip (`FriendlyDamage = true`, `weapons_prime.lua` line 142).
8. **Wesnoth's poison can't kill.** "Poison cannot, of itself, kill a unit" (`data/core/macros/weapon_specials.cfg` line 207). The code floors HP at 1 (`src/actions/heal.cpp` lines 329–332). Undead are immune (`data/core/macros/traits.cfg` lines 57–63).
9. **Stacking is capped by board space or cost, not by a number.** Duelyst creep is permanent, so it hits again every turn. Darkspine Elemental doubles it (`core/faction4.coffee` line 529). Abyssal Tormentor, a 6-mana card, makes it equal to the number of creep tiles (`coreshatter/faction4.coffee` lines 257–266). ITB fire is a yes/no status. A second fire source does not add damage.
10. **Recommendation (inference):** fire 1 per turn for 2 turns, shock 1 (electric robots 2), bump 1, fuel explosion 1 to adjacent. Water sinks Heavy robots only. Player-made hazards cannot take a player unit below 1 HP. Early robots that push deal 1, not 2. Details in section 6.

## 1. Into the Breach

### Damage-over-time and chip damage

| Effect | Damage | Who it hits | Source (`scripts/`) |
|---|---|---|---|
| Fire (status) | 1 at start of every turn, until repaired | Everyone (mechs can repair) | `text_tooltips.lua` 84; `text.lua` 139 |
| Fire tile / forest fire | Sets units on fire | Everyone | `text_tooltips.lua` 3–4 |
| Lava | Like water; survivors catch fire | As water | `text_tooltips.lua` 27, 83 |
| A.C.I.D. | ×2 weapon damage only | Everyone | `text_tooltips.lua` 89 |
| A.C.I.D. tile / pool | Applies A.C.I.D. | Everyone | `text_tooltips.lua` 26, 28 |
| Bump (blocked push) | 1 to both units | Everyone | `text.lua` 166 |
| Blocking a Vek spawn | 1 to the blocker | Everyone | `text_tooltips.lua` 35, 90 |
| Explosive Decay | 1 to adjacent tiles on death | Everyone near | `text_tooltips.lua` 70 |
| Storm Generator smoke | 1 per turn (upgrade 2) | Enemies only | `weapons_passive.lua` 51, 67; `text_tooltips.lua` 7 |
| Psion Tyrant | 1 to all player units per turn | Player only | `text_tooltips.lua` 64 |
| Acid snowstorm | 1 + A.C.I.D. on marked tiles | Everyone | `missions/snow/mission_snowstorm.lua` 33–35 |

Non-damage effects (smoke, freezing, ice, conveyors, tides) are in `text_tooltips.lua` lines 6–52.

### Instant kills (`DAMAGE_DEATH`)

| Hazard | Rule | Source |
|---|---|---|
| Water | Most non-flying enemies die. Mechs (`Massive`) survive. | `text_tooltips.lua` 25, 80; `text.lua` 163 |
| Chasm | "Ground units pushed in will be destroyed" | `text_tooltips.lua` 20 |
| Old Earth mine | Unit that stops there is killed | `items.lua` 1–5; `text_tooltips.lua` 38 |
| Air strike | Target plus 4 neighbours | `missions/grass/mission_airstrike.lua` 11, 14–19 |
| Lightning | 4 tiles per turn | `missions/sand/mission_lightning.lua` 13, 40 |
| Falling rock, tentacles, volcanic projectile | Kill any unit | `text_tooltips.lua` 43–46 |

Notes:
- 28 pawns are `Massive` (`pawns.lua`), including all mechs. 12 are flying. Both groups are immune to drowning.
- Every timed hazard marks its tile one turn early: "will affect this space next turn" (`text_tooltips.lua` line 105).
- The developers put clarity first. Justin Ma: "we would sacrifice cool ideas for the sake of clarity every time" ([Game Developer](https://gamedeveloper.com/design/-i-into-the-breach-i-dev-on-ui-design-sacrifice-cool-ideas-for-the-sake-of-clarity-every-time-)).
- Pilots can turn fire off or invert it: Cauterize ("Fire heals instead"), Rockman ("Immune to Fire") (`text_tooltips.lua` lines 120, 131). Flame Shielding makes all mechs immune (`text_weapons.lua` line 157).

### Relative to HP

| Hazard | Mech (2–3) | Normal Vek (median 3) | Boss (5–7) |
|---|---|---|---|
| Fire tick (1) | 33–50% | 33% | 14–20% |
| Bump (1) | 33–50% | 33% | 14–20% |
| Water | Survives | Dies (unless flying) | Immune |
| Chasm, mine, strike | Dies | Dies | Dies |

## 2. Duelyst

All tiles are in `app/sdk/cards/factory/misc/tiles.coffee`. Text is in `app/localization/locales/en/modifiers.json`.

| Tile | Effect | Who | Source |
|---|---|---|---|
| Shadow Creep | 1 damage at end of owner's turn, permanent | Enemy minions and generals | `modifierStackingShadows.coffee` 25, 84–100; `modifiers.json` 86 |
| Hallowed Ground | Heals 1 at end of owner's turn | Allies | `modifierHallowedGround.coffee` 23, 47–54; `modifiers.json` 58 |
| Mana Spring | +1 mana this turn | Collector | `modifierCollectableBonusMana.coffee` 18; `gameSetup.coffee` 93–95 |
| Primal Flourish | Allied minions on it get "Grow: +2/+2" | Allies | `modifierPrimalTile.coffee` 27; `modifiers.json` 74 |

Creep makers and boosters (`app/sdk/cards/factory/`):

| Card | Cost | Effect | Source |
|---|---|---|---|
| Shadow Nova (spell) | 4 | Creep in a 2×2 area | `core/faction4.coffee` 901–914 |
| Darkspine Elemental (1/4) | 2 | Doubles friendly creep damage | `core/faction4.coffee` 499, 525–529 |
| Abyssal Tormentor (6/6) | 6 | Creep damage = number of friendly creep tiles | `coreshatter/faction4.coffee` 257–266 |

- Creep (1) is 33% of a median 3-HP minion and 4% of a 25-HP general (`hp-scale.md`).
- Creep is one-sided. It hurts only enemies, so it never punishes its owner.
- Stacking is uncapped in code. The cap is cost: the scaling card takes 6 mana and a board full of creep.

## 3. Wesnoth

Wesnoth has no damaging terrain. Terrain changes the chance to be hit and the movement cost.

| Effect | Value | Source |
|---|---|---|
| Poison | −8 HP per turn, floors at 1 HP | `data/game_config.cfg` 19; `src/actions/heal.cpp` 324–332 |
| Poison immunity | Undead trait | `data/core/macros/traits.cfg` 57–63 |
| Village heal | +8 per turn, or cures poison | `data/core/terrain.cfg` 1787 (`heals=8`), 3429 |
| Rest heal | +2 if the unit didn't move or fight | `data/game_config.cfg` 20; `heal.cpp` 311–313 |
| Regenerates | +8 per turn | `data/core/macros/abilities.cfg` 98–99 |
| Slow | Halves damage dealt, doubles move cost, one turn | `weapon_specials.cfg` 84 |
| Shallow water (smallfoot) | 80% chance to be hit (flat 60%), 3 move | `data/core/units.cfg` 419, 435, 438 |

- Poison 8 is 24% of a median level-1 unit (33 HP). It is about one normal hit per turn.
- Healing 8 exactly cancels poison 8. A village is the counter to poison.
- Wesnoth has no pushes, so it has no drowning. Water is a bad place to stand, not a death trap.

## 4. Cross-game comparison

| Question | ITB | Duelyst | Wesnoth |
|---|---|---|---|
| DoT per turn ÷ typical HP | 1 ÷ 3 = 33% | 1 ÷ 3 = 33% | 8 ÷ 33 = 24% |
| DoT tick vs normal hit | 1 vs 1 (equal) | 1 vs 3 (less) | 8 vs 7–9 (equal) |
| Instant-kill hazards | Many, all telegraphed | None | None |
| Who dies instantly | Filtered by type; player mechs survive water | – | – |
| Hits both sides? | Mostly yes; a few one-sided | Enemy only | Yes (poison is an attack) |
| Can DoT kill? | Yes | Yes | No (floor at 1) |
| Multipliers apply? | No (A.C.I.D., armor are weapon-only) | Yes (Darkspine ×2) | – |
| Stack limit | Status is yes/no | Cost and board space | Poison is yes/no |

Patterns:
- **Telegraph** every lethal tile one turn early (ITB).
- **Filter** instant kills by unit type, and pick the filter so the player's core units survive (ITB water and `Massive`).
- **One-sided** hazards for the player's own tools (Duelyst creep, ITB Storm Generator).
- **Floor at 1 HP** so a hazard alone never finishes a unit (Wesnoth poison).
- **Yes/no statuses** so a second fire source adds nothing (ITB, Wesnoth).

## 5. hello1 constraints that matter

- Hazards come only from the player's parts, plus water. So the player always caused the hazard. A punishing friendly-fire rule would feel like a trap more than a puzzle **(inference)**.
- Animals die permanently. ITB mechs come back, so ITB can afford symmetric fire. hello1 needs a softer rule for its own units **(inference)**.
- The check from `hp-scale.md`: early robot hits are at most 2, and animals have 3 HP. So 2 + any 1-damage hazard = 3 = dead. The rules below exist to block that.

## 6. Recommended tile and effect numbers (inference)

All of this section is **(inference)** from sections 1–4.

| Tile / effect | Robots | Player units (human, animals) | Notes |
|---|---|---|---|
| Ground | – | – | – |
| Wall bump | 1 | 1 | Also 1 to the unit bumped into, both sides (ITB rule) |
| Water, Heavy robot pushed in | **Instant kill** | – | The ITB water filter. Telegraph with a skull icon. |
| Water, other robot or walker pushed in | 1, bounced out | 1, bounced out | Counts as a bump. |
| Water, boss | 1, bounced out | – | Bosses never sink (ITB boss rule). Elite Heavy still sinks. |
| Swimmer in water | – | +1 to its attacks | No damage from plain water. |
| Fire | 1 per turn, 2 turns (2 total) | 1 per turn, floors at 1 HP | Yes/no status. Re-igniting resets to 2 turns, never 2 per turn. |
| Fuel robot ignited | **Instant kill** (itself) | – | Then 1 to each adjacent tile, both sides. Floors at 1 for player units. |
| Shocked water | 1; Electric robot 2 | 1 to swimmers, floors at 1 | Each unit is hit once per spread, however many paths reach it. |
| Ruined building | – | – | Cover / perch only. |
| Habitat | – | – | No damage. |

Core rules:
1. **Hazard damage is flat.** Weaknesses and multipliers apply only to weapon hits. Electric robots taking 2 from shock is a fixed table entry, not a multiplier (ITB A.C.I.D./armor rule).
2. **Player-made hazards cannot take a player unit below 1 HP** (Wesnoth poison rule). They still kill robots. The board stays one rule set; only the floor differs.
3. **No unit takes more than 3 from one player action** (the combo cap in `hp-scale.md`). Example: fire tick 1 + Flame-Thrower-style "burning bonus" 2 = 3.
4. **Hazard ticks resolve after robot attacks**, at the end of the robot turn. A wounded animal then never loses its last HP to a tick before the player can act.
5. **In early battles, robot attacks that push deal 1, not 2.** Strong 2-damage robot attacks don't push.
6. **Only Heavy robots and ignited Fuel robots die instantly to tiles.** Animals and the human never do.

### Early-run lethality check (full-HP animal, 3 HP)

| Worst case in one enemy turn | Damage | Result |
|---|---|---|
| Strong hit (2) + standing in fire (tick floored) | 2, then 1 → floor holds | Survives at 1 |
| Push attack (1) + pushed into water (1) | 2 | Survives at 1 |
| Push attack (1) + wall bump (1) | 2 | Survives at 1 |
| Push attack (1) + bumped into another unit (1) | 2 | Survives at 1 |
| Strong hit (2) + adjacent fuel explosion (1) | 2, then 1 → floor holds | Survives at 1 |
| Swimmer: strong hit (2) + shocked water (1) | 2, then 1 → floor holds | Survives at 1 |
| Two separate robots, 2 + 2 | 4 | Dies. Out of scope for this check. The solver check in `balancing-tactics-games.md` should cover it. |

Sturdy animals (4) and the human (6) pass with room to spare. Calmed wild animals (2) do not: push 1 + water 1 = 2 kills one. That is fine for a temporary helper, but it should show a lethal warning.

Robots under the same numbers:
- Light (1–2): dies to any bump, water bounce or one fire tick. Fire alone clears it in 2 turns.
- Fuel / Electric (2–3): two fire ticks or one shock (2) plus a bump finishes an Electric robot.
- Heavy (4): water is the clean answer. Fire alone only removes half its HP.
- Boss (6–7): hazards are chip damage, 14–17% per tick, like ITB fire on bosses.

## 7. Unverified

- **When ITB fire ticks.** The tooltip says "start of every turn" (line 84). I did not confirm whether that means each side's turn or only one.
- **Whether a mech in a chasm is "disabled" or permanently lost.** The tooltip says "destroyed" (line 20). The engine code is not in Lua.
- **LearnBreach fidelity.** It may be a development or Advanced Edition snapshot (see `hp-scale.md`).
- **Wesnoth deep water.** `smallfoot` lists no `deep_water` cost (`units.cfg` 417–432). I assume the default is impassable but did not trace it in C++.
- **ITB developer statements on hazards.** I found only the general clarity quote. The GDC 2019 postmortem may say more; I did not watch it.
- **Divinity: Original Sin surfaces, XCOM fire/acid, Fights in Tight Spaces.** Not researched. I found no first-party numbers quickly, so they have no section.

## Sources

All file paths are cited inline. Web: LearnBreach (https://github.com/2147211690/LearnBreach) and the Game Developer article above.
