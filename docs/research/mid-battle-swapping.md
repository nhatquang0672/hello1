# Swapping animals mid-battle

Research notes for hello1. Researched 2026-11-06. Claims are cited with a URL or a local path. Anything marked **(inference)** is my own reasoning and not sourced. Several wikis (Bulbapedia, Final Fantasy Wiki, GamerGuides) refused automated fetches; where a claim rests only on a search-result snippet, it says so.

## 1. Summary: takeaways for hello1

1. Every game that allows swapping or reinforcing mid-battle puts a price on it. The price is usually the turn (Pokémon, Darkest Dungeon), a place (Wesnoth's keep, Wargroove's buildings), or arrival "exhausted" with no action that turn (Wesnoth, Duelyst). Free swapping (Final Fantasy X) shows up in a menu RPG with no board positions, not in grid tactics ([Game8 snippet](https://game8.co/games/Final_Fantasy_X/archives/270677); `wesnoth/src/actions/create.cpp` lines 626–631; `duelyst/app/sdk/modifiers/modifierFirstBlood.coffee`).
2. Unit count is the dangerous lever. In Into the Breach one extra enemy can make a board unsolvable against "your limited 3 actions" ([ITB AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic); `docs/research/balancing-tactics-games.md`). Any swap rule that ever puts a third animal on the board changes the action economy, not just the options **(inference)**.
3. Into the Breach has no swapping. A destroyed mech sits out the rest of the battle and comes back next battle with an AI pilot. Subset treats losing a unit as a tool the player may choose, not as a failure state ([Game Developer](https://www.gamedeveloper.com/design/reimagining-failure-in-strategy-game-design-in-i-into-the-breach-i-)). hello1's current "fallen animal is out for this battle" rule matches this.
4. The most common middle ground is "replace on fall": Pokémon lets a new Pokémon come in after a faint without losing a turn ([Bulbapedia Recall, snippet](https://bulbapedia.bulbagarden.net/wiki/Recall)). It adds a comeback option without opening up a per-turn choice **(inference)**.
5. Swapping cuts against build commitment. One indie developer notes that "the strategy in picking a set party order would be lost if you just picked on the fly" ([itch.io devlog, snippet](https://itch.io/devlog/1024303/general-update-battle-change.amp)). With 2 animals and 1–2 parts each, the pick at run start is a large part of hello1's identity **(inference)**.

## 2. How comparable games handle it

**Pokémon (switch costs the turn).** Switching happens before moves in the turn order ([Bulbapedia Priority, snippet](https://bulbapedia.bulbagarden.net/wiki/Priority)). A voluntary switch uses the trainer's action for the round, so the incoming Pokémon "will 'lose' a turn". A replacement sent out after a faint loses no turn ([Bulbapedia Recall, snippet](https://bulbapedia.bulbagarden.net/wiki/Recall)). The usual numeric priority (+6) comes from user posts only ([PokéBase](https://pokemondb.net/pokebase/119034/how-much-priority-does-switching-have)). Moves like U-turn attack and switch in one action ([Bulbapedia U-turn](https://bulbapedia.bulbagarden.net/wiki/U-turn%20(move))). Pokémon's switching works because of hidden simultaneous choices and type matchups. hello1 has neither: its robots telegraph and its information is perfect **(inference)**.

**Final Fantasy X (free swap).** The active party is three of seven characters ([Matchstick Eyes](https://matchstickeyes.com/2014/05/18/i-choose-you-combat-in-final-fantasy-x)). On a character's turn the player can swap that character for one from the back row ([Game8, snippet](https://game8.co/games/Final_Fantasy_X/archives/270677)). Reviewers call it the system's biggest change and note that players rotate everyone through to share AP ([HonestGamers](https://www.honestgamers.com/793/playstation-2/final-fantasy-x/review.html), snippet). Swapping acts as a free "answer key" for each enemy, which FFX's fights are designed around. There is no board for positions or threats to land on **(inference)**.

**Darkest Dungeon (no reserves, but shuffling costs a turn).** Heroes stand in ranks 1–4 and get one turn per round by default ([DD wiki, Combat Mechanics](https://darkestdungeon.wiki.gg/wiki/Combat_Mechanics_(Darkest_Dungeon))). Skills that move the user let a hero avoid having to "forfeit a turn of attacks in order to regain position" ([DD wiki, Move](https://darkestdungeon.wiki.gg/wiki/Move)). In DD2, moving a hero in combat counts as an action and takes the turn ([Gamepressure](https://www.gamepressure.com/newsroom/darkest-dungeon-2-how-to-change-party-order-in-combat-and-outside/z555f5), snippet). The roster is chosen outside combat, and the party can't swap in a reserve mid-fight (I found no rule allowing it; see Unverified). The interesting decisions come from position inside a fixed party, not from who is in the party **(inference)**.

**Fire Emblem (Rescue, Pair Up).** Rescue (Thracia 776 to Radiant Dawn) lets a unit carry a lighter ally. Carrying halves the rescuer's skill and speed (more stats in Thracia), and "Rescuing a unit ends the rescuer's turn". "Travelers cannot perform any actions themselves". A dropped traveler counts as already moved that turn. If the rescuer dies, the traveler is released ([Fire Emblem Wiki](https://www.fireemblemwiki.org/wiki/Rescue_(command))). Awakening's Pair Up stacks two units on one tile. The lead gets stat bonuses from the partner. "Switch" swaps lead and support, but the pair can't use remaining movement after switching ([Fire Emblem Wiki, Dual](https://www.fireemblemwiki.org/wiki/Dual); [GamerGuides, snippet](https://www.gamerguides.com/fire-emblem-awakening/guide/intro-and-gameplay/gameplay/support-and-pairing-up)). Both are on-board swaps between units already deployed, not a bench, and both cost movement or the turn.

**Wesnoth (recruit and recall from a keep).** Source code, local repo:
- Recruiting or recalling requires a leader standing on a keep: "You must have a leader on a keep who is able to recall that unit." The unit appears on a vacant castle tile connected to that keep (`wesnoth/src/actions/create.cpp` lines 258–262, 346–352, 388–392).
- Units arrive with 0 movement and 0 attacks unless a scenario says otherwise (`create.cpp` lines 626–631; `full_movement = false` default in `src/actions/create.hpp` line 162).
- Recall costs 20 gold by default (`wesnoth/data/game_config.cfg` line 21), with a per-unit override (`create.cpp` line 754).

So reinforcement is limited by place (the leader has to walk back to the keep), price (gold), and tempo (the new unit can't act this turn) **(inference)**. That makes it a visible, plannable decision.

**Duelyst (summoning next to friendly units).** A minion can be summoned onto tiles around any allied unit, including the general (`duelyst/app/sdk/board.coffee`, `getValidSpawnPositions`, lines 302–316). Airdrop is a keyword for the exception: "Summon this minion anywhere on the battlefield." Rush is the keyword for "Can move and attack immediately" (`duelyst/app/localization/locales/en/modifiers.json` lines 120–123). Rush's code refreshes exhaustion on the summon turn, which shows that summoned minions start exhausted by default (`duelyst/app/sdk/modifiers/modifierFirstBlood.coffee`). Every summon also costs mana.

**Wargroove (production).** Gold buys units at buildings, and "Each structure can only be used to recruit a unit once per turn" ([Wargroove Wiki, Recruit](https://wargroovewiki.com/Recruit)). Units appear on a free tile next to the building ([Wargroove Wiki, Barracks](https://wargroovewiki.com/Barracks); [Game Informer](https://gameinformer.com/2019/02/01/10-ways-wargroove-changes-the-way-you-play-advance-wars)). This is an economy game. Production is the core loop, and it doesn't carry over to a fixed small squad **(inference)**.

**Into the Breach (fixed three).** The squad is three mechs ([Wikipedia](https://en.wikipedia.org/wiki/Into_the_Breach)). A destroyed mech comes back next battle with an AI pilot that can't earn XP. Davis: "if they die you get punished with the pilot death". Ma says Subset is "requiring players to unlearn something that's been taught by almost every other strategy game". They heal mechs fully after each battle "so you can really use their health bar as a tool during the battle". Ma also says the worst outcome is a situation with "no options whatsoever to resolve" ([Game Developer](https://www.gamedeveloper.com/design/reimagining-failure-in-strategy-game-design-in-i-into-the-breach-i-)). Multiple squads were among the 60% of plans that were cut ([search summary of Kotaku interview](https://kotaku.com/1825719534); not fetched).

**Banner Saga (fixed deployment).** The player picks up to 6 heroes and places them before the fight ([Banner Saga Wiki](https://bannersaga.fandom.com/wiki/Getting_Started), snippet). Turns alternate one unit per side, so a smaller side gets more turns per unit. When one side has a single unit left, "Pillage" lets the other side act with everyone ([Banner Saga Wiki, Combat](https://bannersaga.fandom.com/wiki/Combat), snippet). It has no mid-battle swapping. It handles losses through turn order instead.

**Monster Train (deploy during battle, with capacity).** Units cost Ember and take up Capacity on a floor (default 5), and they stay until they die ([BlueStacks guide](https://www.bluestacks.com/blog/game-guides/monster-train/mstn-beginners-guide-en.html), snippet; [Steam discussion](https://steamcommunity.com/app/1102190/discussions/0/3117032860236072824)). Mid-battle deployment is the whole game here, limited by a per-turn resource and a per-floor cap.

**Slay the Spire** has no units to swap. It's included only as the deck-builder baseline: build identity is set between fights and expressed inside them ([Game Developer](https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder)).

## 3. Design trade-offs

I found little developer writing about mid-battle swapping specifically (see Unverified). The points below combine sourced ITB principles with **(inference)**.

- **Clarity.** Subset "would sacrifice cool ideas for the sake of clarity every time" ([Game Developer, UI](https://www.gamedeveloper.com/design/-i-into-the-breach-i-dev-on-ui-design-sacrifice-cool-ideas-for-the-sake-of-clarity-every-time-)). A benched animal is off-board information: the player has to remember its parts and cooldowns. That cuts against a readable board **(inference)**.
- **Options and analysis paralysis.** A free swap multiplies the choices each turn by (bench size + 1) for each animal. A deterministic puzzle is already a search over moves, so every option widens the tree the player has to check **(inference)**.
- **Balance and counter-picking.** Free swapping lets the player bring the right animal for each telegraphed threat. Because hello1 shows threats in advance, a free swap would answer most of them and flatten encounter design. FFX can afford this because its fights expect it **(inference)**.
- **Action economy.** The ITB notes tie solvability to unit count ([AI notes](https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic)). A swap that never raises the number of animals on the board keeps the action count at 3. Reinforcements that add a body break the tuning **(inference)**.
- **Run identity.** "The strategy in picking a set party order would be lost if you just picked on the fly" ([itch.io devlog, snippet](https://itch.io/devlog/1024303/general-update-battle-change.amp)). The same devlog limited swaps to before any action that turn, after swapping in units that had already acted caused bugs. That is a real edge case: does an incoming animal inherit the outgoing one's spent action?
- **Losses as tools.** ITB makes losing a unit survivable so players can spend it ([Game Developer](https://www.gamedeveloper.com/design/reimagining-failure-in-strategy-game-design-in-i-into-the-breach-i-)). A replace-on-fall bench makes sacrificing an animal cheaper still. That can be good (bolder plays) or bad (sacrifice becomes routine) **(inference)**.
- **Implementation.** Swapping adds state: bench cooldowns (do they tick?), entry tiles, blocked entry, undo. The brute-force solvability check suggested in `balancing-tactics-games.md` also has to model it **(inference)**.

## 4. Variants between "never" and "free swap"

| Variant | Example | Cost or limit |
|---|---|---|
| Never mid-battle; swap between battles | ITB, Banner Saga, hello1 proposal | none in battle |
| Swap costs the incoming unit's turn | Pokémon voluntary switch | turn lost |
| Replace only on fall, no turn lost | Pokémon after faint | only after a loss |
| Swap only at a place | Wesnoth keep, Wargroove buildings | leader must be there, arrives with 0 moves |
| Arrive next to a friendly unit, exhausted | Duelyst summon | mana, no action that turn |
| On-board carry/stack | FE Rescue, Pair Up | ends turn, stat penalty |
| Free swap on the unit's turn | FFX | none |
| Per-turn resource plus board cap | Monster Train | Ember, floor Capacity |

Other possible hybrids **(inference)**: a bench of 1 with a once-per-battle swap; a swap that triggers a part's cooldown; a swap where the incoming animal enters on a telegraphed tile at the start of the next turn, so robots can react to it.

## 5. Options for hello1 (inference)

All of this is **(inference)**. The choice is the user's.

**A. Keep the current proposal (no mid-battle swap).** Rescue swaps between battles only, and a fallen animal is out for the rest of the battle.
- Pros: simplest rules, perfect information on the board, strongest run identity, and no solver or UI changes. It matches ITB's model.
- Cons: losing an animal early can make the rest of a battle a slog with 2 bodies, which risks Ma's "no options whatsoever" case. Map and spawn design have to absorb that.

**B. Bench of 1, replace on fall.** The player takes a third animal as reserve. When an animal falls, the reserve enters at a marked tile next to the human at the start of the next player turn, exhausted, with full cooldowns. It enters once per battle.
- Pros: never more than 2 animals on the board, so the action economy stays intact. It softens bad losses and gives the reserve pick some build interest.
- Cons: sacrificing an animal becomes cheaper. Off-board information (the reserve's parts) needs a clear panel. A third animal choice also adds to run-start decisions.

**C. Van tile (Wesnoth-style keep).** Some maps have a van tile. If the human ends a move on it, the human can spend its action to swap one animal for the benched one. The newcomer appears next to the van with no action that turn.
- Pros: a visible, telegraphable decision. It costs the leader's tempo and position, which makes it a real trade-off, and it adds map variety.
- Cons: most rules and state of the three options. It risks becoming a counter-pick tool, and it pulls the human, whose death ends the run, toward a fixed spot.

To check whichever option is chosen: if B or C, does a benched animal's cooldown tick? Can the entry tile be blocked? Can the incoming animal act on the turn it enters? (Wesnoth and Duelyst both default to "no".)

## 6. Unverified claims

- The exact Pokémon switching rules: Bulbapedia returned 403, so these come from search snippets only. The +6 priority number and trapping rules are unconfirmed.
- Whether the incoming FFX character keeps the outgoing character's CTB slot or loses time. The Final Fantasy Wiki returned 402 and GamerGuides returned 403.
- That Darkest Dungeon (1) has an in-combat "swap with adjacent hero" that uses the turn, and that reserves can't come in mid-fight. Only secondary snippets and the Move page's wording support this.
- That Into the Breach has missions with player-controlled bonus units such as tanks. I couldn't confirm "deployable tanks" in any source.
- That a destroyed ITB mech stays on the board as a wreck for the rest of the mission. This is widely reported by players but wasn't in the fetched article.
- Whether Wargroove or Advance Wars units can act on the turn they are built. The Wargroove Wiki page didn't say.
- That Wesnoth's recall list carries surviving units between campaign scenarios. This is widely known, but I didn't check it in source.
- I found no developer talk or postmortem that analyses mid-battle swapping trade-offs directly. Section 3 rests on ITB principles and inference.

## 7. Sources

- Bulbapedia, Priority: https://bulbapedia.bulbagarden.net/wiki/Priority (snippet only)
- Bulbapedia, Recall: https://bulbapedia.bulbagarden.net/wiki/Recall (snippet only)
- Bulbapedia, U-turn: https://bulbapedia.bulbagarden.net/wiki/U-turn%20(move) (snippet only)
- PokéBase on switch priority: https://pokemondb.net/pokebase/119034/how-much-priority-does-switching-have
- Game8, FFX battle basics: https://game8.co/games/Final_Fantasy_X/archives/270677 (snippet only)
- Matchstick Eyes, FFX combat: https://matchstickeyes.com/2014/05/18/i-choose-you-combat-in-final-fantasy-x
- HonestGamers, FFX review: https://www.honestgamers.com/793/playstation-2/final-fantasy-x/review.html
- Darkest Dungeon Wiki, Combat Mechanics: https://darkestdungeon.wiki.gg/wiki/Combat_Mechanics_(Darkest_Dungeon)
- Darkest Dungeon Wiki, Move: https://darkestdungeon.wiki.gg/wiki/Move
- Gamepressure, DD2 party order: https://www.gamepressure.com/newsroom/darkest-dungeon-2-how-to-change-party-order-in-combat-and-outside/z555f5
- Fire Emblem Wiki, Rescue: https://www.fireemblemwiki.org/wiki/Rescue_(command)
- Fire Emblem Wiki, Dual: https://www.fireemblemwiki.org/wiki/Dual
- GamerGuides, Awakening Pair Up: https://www.gamerguides.com/fire-emblem-awakening/guide/intro-and-gameplay/gameplay/support-and-pairing-up
- Wargroove Wiki, Recruit: https://wargroovewiki.com/Recruit ; Barracks: https://wargroovewiki.com/Barracks
- Game Informer, Wargroove vs Advance Wars: https://gameinformer.com/2019/02/01/10-ways-wargroove-changes-the-way-you-play-advance-wars
- Game Developer, "Reimagining failure in Into the Breach": https://www.gamedeveloper.com/design/reimagining-failure-in-strategy-game-design-in-i-into-the-breach-i-
- Game Developer, ITB UI and clarity: https://www.gamedeveloper.com/design/-i-into-the-breach-i-dev-on-ui-design-sacrifice-cool-ideas-for-the-sake-of-clarity-every-time-
- ITB programmer's AI notes: https://docs.google.com/document/d/1OPRNzAVQNgPtP35HzhV2ZzEqE1eCdXbWA7lEZXDdMHQ/mobilebasic
- Wikipedia, Into the Breach: https://en.wikipedia.org/wiki/Into_the_Breach
- Kotaku interview (not fetched): https://kotaku.com/1825719534
- Banner Saga Wiki: https://bannersaga.fandom.com/wiki/Getting_Started ; https://bannersaga.fandom.com/wiki/Combat (snippets)
- Monster Train: https://www.bluestacks.com/blog/game-guides/monster-train/mstn-beginners-guide-en.html ; https://steamcommunity.com/app/1102190/discussions/0/3117032860236072824
- Game Developer, Slay the Spire data: https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder
- itch.io devlog on mid-battle swaps: https://itch.io/devlog/1024303/general-update-battle-change.amp (snippet only)
- Local: `wesnoth/src/actions/create.cpp`, `wesnoth/src/actions/create.hpp`, `wesnoth/data/game_config.cfg`
- Local: `duelyst/app/sdk/board.coffee`, `duelyst/app/sdk/modifiers/modifierFirstBlood.coffee`, `duelyst/app/localization/locales/en/modifiers.json`
- Local: `docs/research/balancing-tactics-games.md`
