# Enemy AI in tactics games

Research notes for hello1. Researched 2026-11-06. Claims are cited with a URL or a local path. Anything marked **(inference)** is my own reasoning and not sourced. This file extends the "Enemy AI" part of `balancing-tactics-games.md` and does not repeat it. Local repo paths are relative to `game-projects/`.

## 1. Summary

1. Into the Breach's attack scoring is a short Lua function. Every tile an attack hits adds a fixed weight: enemy +5, building +5, friendly −2, empty 0. Weapons can override the weights or the whole function (`scripts/global.lua` lines 265–269 and 400–441 in a [copy of the game scripts](https://github.com/2147211690/LearnBreach/blob/main/scripts/global.lua)).
2. ITB scores the end tile separately from the attack. Danger tiles return −10 and corners −2. Melee units score +5 next to a target and otherwise get a "move closer" gradient. A very bad position cancels the attack (`global.lua` lines 437–439 and 446–505).
3. ITB picks randomly among the top-scored options. Difficulty changes how often it takes the best one rather than a "second best" one. Enemies act in queue order, and each one treats tiles already targeted by earlier enemies as dangerous ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)).
4. ITB's AI does not tell a building from a mech. Both are worth 5 ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic); `global.lua` lines 265–268).
5. Duelyst's AI is a fixed priority list plus additive "bounty" weights. It takes lethal attacks first, then moves, then other attacks, then card plays. Each step takes the best-scored option (`duelyst/server/ai/starter_ai.js` lines 290–475; `duelyst/server/ai/scoring/bounty.js`).
6. Duelyst sets difficulty with one number from 0 to 1. Lower values cap kills and summons per turn, make random units skip their attack, and stop units attacking the enemy general. Difficulty ramps up with the player's win count (`starter_ai.js` lines 391–401 and 2619–2628; `duelyst/server/routes/api/me/games.coffee` line 395).
7. Wesnoth's RCA AI is a loop. Each candidate action has a fixed maximum score, and the loop runs the best-scoring action, then re-evaluates everything, until nothing scores above zero. There is no randomness in which action it picks (`wesnoth/src/ai/default/stage_rca.cpp` lines 75–137; `wesnoth/data/core/macros/ai_candidate_actions.cfg`).
8. Wesnoth tunes behaviour with two numbers. `aggression` (default 0.4) discounts the AI's own expected losses. `caution` (default 0.25) penalises leaving good terrain (`wesnoth/src/ai/default/attack.cpp` lines 268–318; `wesnoth/data/ai/utils/default_config.cfg` lines 44 and 58).
9. Forge has no single scoring function. It sorts all playable spells "best first", then asks each spell's API-specific AI class whether to play it, and plays the first one approved. Per-card hints live in card scripts as `AILogic$` parameters (`forge/forge-ai/src/main/java/forge/ai/AiController.java` lines 2546–2670; `SpellApiToAi.java`; `SpellAbilityAi.java` lines 72–166).
10. Forge's randomness and personality come from `.ai` profile files of "chance" percentages, such as `CHANCE_TO_ATTACK_INTO_TRADE`. A `Reckless` profile is mostly those numbers turned up (`forge/forge-gui/res/ai/Default.ai`, `Reckless.ai`).
11. Of the four games, only Forge searches ahead, and that layer is optional. It replays the top few heuristic choices in game copies and keeps the heuristic choice unless another one is clearly better (`forge/forge-ai/src/main/java/forge/ai/simulation/FlatMonteCarloPicker.java` lines 30–51).

## 2. Into the Breach

**Sources.** The [AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic), plus the game's plain-Lua scripts, read from a [third-party copy](https://github.com/2147211690/LearnBreach). Mods hook the same names (`ScorePositioning`, `Skill.GetTargetScore`, `ScoreFriendlyDamage = -2`), which supports the copy ([Lemonymous mods](https://github.com/Lemonymous/ITB-LemonymousMods), [ITB-Hotseat](https://github.com/Lemonymous/ITB-Hotseat)). The outer loop (enumerating moves, sorting, the random pick) is not in Lua.

**Attack score (`Skill:GetTargetScore`, `global.lua` lines 378–394).**
- The game builds the attack's effect list for a move tile and a target tile, then scores it.
- A queued (telegraphed) attack is scored on its queued effects. An instant attack is scored on its instant effects. An instant attack scoring below −20 returns −100, with the comment "don't do anything so horrible if it's instant".

**Per-tile weights (`Skill:ScoreList`, lines 400–441; defaults at 265–269).**
- Enemy pawn hit: `ScoreEnemy = 5`. A dead or temporary pawn sets the score to `ScoreNothing`.
- Building hit (powered, damage > 0) or non-grid structure: `ScoreBuilding = 5`.
- Own team hit: `ScoreFriendlyDamage = -2`. A frozen ally that is not already targeted scores +5 instead, because the hit thaws it.
- Time pod hit by an instant effect: return −100.
- Push or move effects on the attacker add `ScorePositioning(end tile)`. If that total is below −5, the function returns it instead, with the comment "blocking friends, dying, etc."
- This matches the notes' "5 pts, 5 pts, and -2 pts", and the notes add that "95% of them" use these weights.

**Overrides.** Weapons override weights or the whole function. The Spider's egg throw scores each tile next to the landing tile: player +5, building +2, empty +1, own team −2 (`scripts/weapons_enemy.lua` lines 776–797 in the same copy). Every pawn also has `PositionScore`, commented "can overwrite the get method to make a custom AI positioning scorer" (`global.lua` line 149).

**Position score (`ScorePositioning`, lines 446–505).** The first matching rule wins:
- Time pod tile −100. Hole −10 for non-flyers.
- Tile already targeted: `pawn:GetDangerScore()`. The default `ScoreDanger` is −10 (line 141).
- Smoke −2. Fire −10 for non-fire units. Spawn tile, mission danger, or mine −10. Water −5 for non-flyers.
- A pawn-specific custom score, if non-zero.
- Corner −2, edge 0.
- Melee: +5 if next to a player unit or a building. Otherwise `max(0, (10 - distance to nearest target) / 2)`.
- Ranged: +5.

**Combining scores.** In the AI notes, any move with a negative position score is dropped. If no attack scores above 0, moves are ranked by position score. Moves are sorted by score and the AI "randomly pick[s] from between 'the best'", sometimes "second best". The notes also add a penalty for repeated positions and targets. This stops a Scorpion from walking back into the same punch every turn ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)). The exact formula that combines the two scores is an image in the doc, so I could not read it.

**What it ignores.** Other enemies' plans, except their targeted tiles. Player weapons. Whether the board is solvable. It "never plan[s] future moves" ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)).

## 3. Forge (Magic: The Gathering)

**Main loop.** `chooseSpellAbilityToPlay` (`AiController.java` line 1596) collects every playable spell and sorts them, commented "put best spells first" (line 2560). The sort key is mana value plus a hand-written priority (`ComputerUtilAbility.java` lines 247–325). The loop plays the first spell whose `canPlaySa` returns `WillPlay` (`AiController.java` lines 949–977, 2638–2670). So "best" means sort order plus a yes/no gate, not a score comparison **(inference)**.

**Per-card logic.** `SpellApiToAi` maps each effect type to an AI class, such as `DealDamage → DamageDealAi` (`SpellApiToAi.java` line 77). There are 155 such classes in `ability/`. Their base class checks restrictions, the card's `AILogic` hint, the phase, and then the effect's own logic (`SpellAbilityAi.java` lines 72–171). In this checkout, 1,816 of 34,134 card scripts contain `AILogic$`, and 4,668 contain `AI:RemoveDeck` (grep of `forge/forge-gui/res/cardsfolder`). A creature's value is 80 + 15 per power + 10 per toughness, plus extras (`CreatureEvaluator.java` lines 38–101).

**Attacking and blocking.** `AiAttackController` compares forces and life totals and sets one `aiAggression` level, from 0 ("stay at home to block") to 5 ("attack at all costs"). A comment admits to "bad 'magic numbers'" (lines 1260–1296). Each creature attacks if it passes that level's rule (lines 1562–1610). Level 4 can come from a dice roll on `chanceToAttackToTrade` (line 1268). `AiBlockController` makes good, gang, trade, then chump blocks. If its life is still in danger, it clears all blocks and retries in a safer order (lines 1147–1217).

**Profiles and randomness.** Profiles are property files: Default, Cautious, Reckless, Hard, Hardest and Experimental (`forge/forge-gui/res/ai/`). Most knobs are percentages, such as `MIN/MAX_CHANCE_TO_RANDOMLY_TRADE_ON_BLOCK=30/70` (`Default.ai` lines 48–49). For one knob, equal min and max make the "AI ... predictable", and a range means it "will randomly pick a value between the two" (`Default.ai` lines 78–81). Reckless mostly turns numbers up, for example `CHANCE_TO_ATTACK_INTO_TRADE=100` (diff against Default). A random profile can be picked per match (`AiProfileUtil.java` lines 47–48).

**Simulation.** Flat Monte Carlo replays "the first few approved plays" in game copies, with hidden cards re-dealt, up to the end of the AI's turn. The heuristic choice gets a bonus "so rollouts only override it for a clear gain". It is "deliberately not a tree search" (`simulation/FlatMonteCarloPicker.java` lines 30–51). It is on by default (`ForgePreferences.java` line 216). A full tree search is opt-in per player (`forge-gui-desktop/.../PlayerPanel.java` lines 418–430). This checkout is recent upstream Card-Forge, so this layer may be newer than most Forge write-ups.

## 4. Duelyst

**Entry point.** Single player builds a `StarterAI` with a difficulty value (`duelyst/server/single_player.coffee` lines 1557–1559). The doc comment calls it a "Simple ai that is able to play faction starter decks" (`starter_ai.js` lines 34–44).

**Action order.** `_findNextActions` returns one action, then is called again. The order is: replace a card, lethal moves and attacks (highest-bounty units first), other moves, non-lethal attacks, card plays, end turn (`starter_ai.js` lines 290–475).

**Target score.** For each reachable target, the score is the damage score plus situational bonuses. These are +15 if the counter-attack won't kill the attacker, +25 for a ranged unit attacking at range, +30 for a backstab, and +47 for a provoke unit facing the general. Targets are sorted high to low (`scoring/utils/utils_findAttackObjectivesAndScoresForUnit.js`). The damage score scales with the target's value. A kill adds a large bonus, minus 6 per point of overkill, and killing the general adds 9999 (`scoring/base/unit_damage.js`; `bounty.js`).

**Position score.** Several small modules are summed. Melee units get −0.5 per tile of distance to their best target. "Evasive" units, meaning ranged units or low-HP generals and buffers, get +3 per tile and keep away (`scoring/position/position_ScoreForCardAtTargetPosition.js`; `position_objective_distanceFromBestObjective.js`; `bounty.js`). All the weights are in one file of named constants (`bounty.js`, 118 lines).

**Per-card logic.** Cards are not code. Each one has a data "intent", such as Tempest = `Burn, amount 2, targets All` (`server/ai/card_intent/faction1.js`). Generic scoring modules turn intents into scores (`scoring/intent/`).

**Difficulty and randomness.** Difficulty is a number from 0 to 1 (`starter_ai.js` line 44). Below 1.0:
- A share of non-general units randomly skip their attack each turn (lines 391–401).
- Kills per turn are capped at `floor(difficulty × 10)`. Summons are capped the same way (lines 407, 917–922).
- Below 0.1, minions never attack the player's general (lines 2499–2502, 2626).
- Avoiding lethal counter-attacks starts at 0.75 (lines 2627–2628).

Difficulty itself ramps with wins: `min(1, win_count / 10)` in single player (`server/routes/api/me/games.coffee` line 395), and `0.2 + 0.8 × min(1, (wins − losses) / 10)` for bots (`server/routes/matchmaker.coffee` line 259). Apart from that, the random calls are mostly tie-breaks over valid positions (`starter_ai.js` lines 823, 1112).

## 5. Wesnoth (RCA AI)

**Loop.** Candidate actions (CAs) are sorted by their fixed `max_score`. Each loop pass evaluates them in order. It stops early once no remaining CA's maximum can beat the best score found so far. It runs the best CA, then starts again, until no CA scores above zero (`wesnoth/src/ai/default/stage_rca.cpp` lines 75–137).

**Priority bands.** The max scores encode a fixed order (`wesnoth/data/core/macros/ai_candidate_actions.cfg`):

goto 200000 › recruit rushers 196000 › retreat injured 192000 › grab villages 191000 › recruitment 180000 › leader to keep 120000 › combat 100000 › healing 80000 › villages 60000 › retreat 40000 › move to targets 20000 › move to any enemy 1000.

**Attack rating (`attack.cpp` lines 268–318).**
- Base value: `chance_to_kill × target_value − avg_losses × (1 − aggression)`. `target_value` is the target unit's cost (line 63).
- If attackers leave better terrain, subtract `caution × exposure × (1 − aggression)`.
- Add a bonus for already-damaged targets and for expected damage dealt.
- A sanity check rejects attacks with high vulnerability, low support, and under 2% chance to kill.
- If the target threatens the AI's leader, the value is multiplied by 5.
- The combat CA keeps the highest-rated attack and runs it if the rating is above 0 (`src/ai/default/ca.cpp` lines 163–231).

**Knobs.** `aggression` 0.4, `caution` 0.25, `leader_aggression` −4.0, `leader_value` 3.0, `village_value` 1.0 (`data/ai/utils/default_config.cfg` lines 44, 58, 60, 63, 101). With `aggression = 1` the AI ignores its own losses entirely (`attack.cpp` line 278).

**Targets for movement.** Movement goals are a list of weighted targets. These are enemies threatening the leader ("threat"), villages, and the enemy leader worth `leader_value` (`src/ai/default/contexts.cpp` lines 137–221).

**Randomness.** I found no random calls in the default CA, attack and move-to-targets files (grep of `ca.cpp`, `attack.cpp`, `aspect_attacks.cpp`, `ca_move_to_targets.cpp`). The AI rates attacks on expected values because Wesnoth's combat rolls dice.

## 6. Cross-game comparison

| | Into the Breach | Duelyst | Wesnoth | Forge |
|---|---|---|---|---|
| Structure | Score every (move, target) pair | Fixed phase order, best per phase | Priority bands, best CA, repeat | Sorted list, first approved |
| Scoring | 3 weights per tile + position table | ~100 named bounty constants | Expected-value formula | Per-API yes/no + evaluators |
| Per-unit logic | Override `GetTargetScore` / position | Data "intents" | Unit cost and stats | `AILogic$` hints + 155 classes |
| Coordination | Earlier attacks mark tiles "dangerous" | Re-plans after every action | Re-evaluates after every action | Re-evaluates per priority |
| Lookahead | None | None (lethal check only) | One attack | Optional rollouts to end of turn |
| Randomness | Random among top scores | Difficulty-driven skips, tie-breaks | None in choice | Profile "chance" rolls |
| Difficulty | Pick best more often | One 0–1 number gating behaviours | `aggression`/`caution` | Profile files |
| Targets | Units = buildings (5 each) | General ×1.4 HP, lethal 9999 | Unit cost; leader threat ×5 | Life total, creature value |

Sources are in sections 2–5.

## 7. Recommendations for hello1 (inference)

All of this section is **(inference)**.

1. **Copy ITB's split.** Use an attack score (sum of per-tile weights) and a position score (a first-match rule table). Drop moves with a negative position score. If no attack scores above 0, rank by position. ITB does this in about 60 lines of Lua.
2. **Keep weights per robot nature, as data.** One table: animal 5, leader 5 or 6, objective 5, other robot −2, nothing 0. A nature overrides a row, such as a "hunter" with leader 8. Write code overrides only for odd weapons, like ITB's Spider.
3. **Weight animals and the leader about equally.** ITB scores buildings and mechs the same, which spreads threats and makes the player choose what to save. A big leader weight makes every robot dive the leader. Duelyst only gives the general a huge weight for a lethal hit.
4. **Randomise only when picking the telegraph.** Pick uniformly among options within N points of the best. N is the difficulty knob, as in ITB. Seed the generator per battle so runs can be replayed. Skip Forge-style percentage rolls, because they hide intent.
5. **Act in queue order and mark claimed tiles as danger.** This spreads robots out for free, and it is the only coordination ITB has.
6. **Add a repeat penalty.** hello1's pushes will create ITB's punch-back loop.
7. **Use dials, not search.** Good dials are the top-N window, "skip the leader on easy" (Duelyst's < 0.1 rule), and a cap on lethal telegraphs per turn. Spawn count stays the main dial (`balancing-tactics-games.md`).
8. **Stay one-ply.** No search and no reading the player's parts. A predictable greedy AI is the puzzle. Board design handles unwinnable states.
9. **Build a score overlay early.** Show each candidate's score on its tile. With three weights, tuning means reading the numbers.

## 8. Unverified

- **ITB Lua copy.** Read from a third-party repo, not a game install. The version is unknown. Mods hook the same names and defaults.
- **Ma: "AI basically just picks from a list of what does the most damage and picks randomly from the top."** Search summaries attribute it to a Game Developer piece. The one I fetched does not contain it. The programmer's notes support the mechanism.
- **Ma: fun "regardless of the AI", and impossibility avoided "less about the AI and more about the map design".** From search snippets of a PC Gamer preview whose body would not load.
- **Slay the Spire.** A third-party analysis of StS 2 code says each monster overrides `GetMove()` using move history, and the intent icon is set after the move is chosen ([DeepWiki](https://deepwiki.com/hanmi255/STS2SourceCode/4.3-monsters-encounters-and-monster-moves)). No first-party source, so no section.
- **XCOM.** Not researched.
- **ITB's score-combining formula** is an image in the AI notes.
- **`scripts/ai.lua`** (from the brief). In the copy I read, scoring lives in `scripts/global.lua`, and there is no `ai.lua`.

## 9. Sources

- ITB AI notes: https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic
- ITB scripts (third-party copy): https://github.com/2147211690/LearnBreach (`scripts/global.lua`, `scripts/weapons_enemy.lua`)
- ITB mods: https://github.com/Lemonymous/ITB-LemonymousMods, https://github.com/Lemonymous/ITB-Hotseat
- Local code: `forge/forge-ai/`, `forge/forge-gui/res/ai/`, `duelyst/server/ai/`, `duelyst/server/routes/`, `wesnoth/src/ai/default/`, `wesnoth/data/core/macros/ai_candidate_actions.cfg`, `wesnoth/data/ai/utils/default_config.cfg`
