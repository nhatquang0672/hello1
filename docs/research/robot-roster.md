# Robot roster and salvage rules

Research notes for hello1. Researched 2026-11-08. Every claim has a local path or URL. **(inference)** marks my own reasoning. This builds on `game-design.md`, `encounter-budget.md`, `enemy-ai.md`, `hp-scale.md`, `tile-damage.md` and `animal-roster.md`, and doesn't repeat them. Local paths are relative to `game-projects/`. ITB paths point to the third-party script copy https://github.com/2147211690/LearnBreach (`scripts/...`), as in the earlier notes.

## Key findings

1. **ITB's 13 Vek use about 10 distinct telegraph shapes.** They are single-tile melee, a 2–3 tile line, a projectile that stops at the first hit, a 1-tile and a 2-tile artillery arc, a charge with push, a web (grapple, then hit), a self-destruct, an egg spawner, and an aura (`text_weapons.lua` lines 15–96).
2. **ITB unlocks enemy types by island and keeps some pairs apart.** Core Vek appear from island 1. "Unique" ones such as Beetle, Digger and Spider come later (`spawner_backend.lua` lines 8–17). Leaper and Scorpion never share a run (lines 24–30).
3. **ITB enemies drop nothing.** Rewards come from objectives and Time Pods, which you "collect with a Mech, or defend until Vek retreat". A pod is "destroyed if damaged or trampled by the enemy" (`text_tooltips.lua` line 30).
4. **ITB's support enemies are fragile and buff everything.** The Psion has 2 HP and flies. It gives all other Vek +1 HP, armor, regeneration, or "explode on death, dealing 1 damage to adjacent tiles" (`pawns.lua` lines 979–1017; `text_weapons.lua` lines 98–111).
5. **Duelyst's "Dying Wish" is the model for on-death effects.** It is defined as "Triggers an effect when this minion dies" (`modifiers.json` line 40). One card even puts "a random legendary artifact from your opponent's Faction into your action bar" (`cards.json` line 573).
6. **Duelyst's parts-boss D3C "cannot be damaged while other D3cepticle parts live".** Each of its four parts carries one keyword (`boss_battles.json` lines 23–35). The MECHAZ0R build combines five keyword parts into one unit (`cards.json` lines 1183–1477).
7. **Wesnoth drops a boss item on its death tile.** A unit picks it up by stepping on it, may leave it, and only some races can take it (`Winds_of_Fate/scenarios/07_Harvest.cfg` lines 612–627; `core/macros/items.cfg` line 74).
8. **Horizon Zero Dawn's machines have "collectable components that could be shot off"**, and the team chose machine shapes so that would work ([PlayStation Blog](https://blog.playstation.com/2017/02/24/the-making-of-horizon-zero-dawns-machines/)).
9. **Kirby lets you "steal their unique attacks and abilities"** and "combine different Copy Abilities" ([Nintendo AU](https://www.nintendo.com/au/games/nintendo-switch/kirby-star-allies/)).
10. **Recommendation (inference):** use 12 robots and 2 bosses, three per nature. Every robot drops its weapon as a part, but how it dies decides whether the part survives. Parts lie on the board and can be trampled. After the battle you keep 1, and the rest become spare parts for team slots.

## 1. Into the Breach

### Vek and Bots (normal tier)

HP and move are from `pawns.lua`. Attack text is from `text_weapons.lua`. Weapon code is in `weapons_enemy.lua`.

| Enemy | HP / move | Attack (shape) | Source lines | Answer it forces |
|---|---|---|---|---|
| Scorpion | 3 / 3 | "Web the target, preparing to stab it" (melee + grapple) | pawns 659; text 15; Web=1 at weapons 38 | Free the webbed unit or kill the Scorpion |
| Leaper | 1 / 4 | Web then bite, 3 damage | pawns 725; text 51 | Kill the fragile, fast threat first |
| Hornet | 2 / 5, flies | "Stab the target" (melee) | pawns 927–931; text 27 | Flyers ignore water. Damage it |
| Firefly | 3 / 2 | Projectile, first unit hit | pawns 690; text 57; weapons 421 | Body-block the line |
| Scarab | 2 / 3 | "Lob an artillery shot at a single tile" | pawns 789; text 39 | Walls don't help. Move off the tile |
| Crab | 3 / 3 | "Artillery attack on 2 tiles" | pawns 826; text 33; weapons 216 | Clear two tiles |
| Beetle | 4 / 2 | "Charge forward to damage and push" | pawns 757; text 63 | Sidestep so it rams something else |
| Digger | 2 / 3 | Builds a rock wall, then hits adjacent tiles | pawns 895; text 69 | Area melee plus new terrain |
| Burrower | 3 / 4, unpushable | "Slam against 3 tiles in a row". Hides "after taking one or more damage" | pawns 1111–1119; text 21; tooltips 62 | You can't push it. Kill it in one turn |
| Centipede | 3 / 2 | Goo that applies A.C.I.D. nearby | pawns 861; text 45 | Debuff area |
| Blobber → Blob | 3 / 2 → 1 / 0 | Throws a blob that will "explode, killing itself… Kill it first to stop it" | pawns 575, 610; text 75–81; weapons 711 | Kill the bomb before it acts |
| Spider | 2 / 2 | Throws an egg that "hatches into a Spiderling" | pawns 1024; text 87–96 | Kill the egg or block the spawn |
| Psion | 2 / 2, flies | Passive aura on all Vek | pawns 979–1017; text 98–111 | Kill the support first |
| Snow Bots (Cannon, Artillery, Laser, Mine) | 1 / 3 (Mine-Bot move 0) | Burning shot; rockets at 3 tiles; "piercing beam, damage reduced by range"; freeze mine | pawns 1179–1297; text 942–961 | 1 HP, so one hit kills each |

- **Alphas** are the same attack with about +2 HP and more damage, for example Scorpion 3 → 5 HP (`pawns.lua` 659, 674). Some gain a wider shape: the Alpha Hornet stabs "2 tiles in front" (`text_weapons.lua` line 30).
- **Bosses** reuse a Vek attack with more HP. One is a support boss: the Bot Leader has 5 HP, is `Massive`, and its `BossHeal` heals and adds a shield (`missions/bosses/bot.lua` lines 28–50).
- **When enemies appear:** Core is Firefly, Hornet, Scarab, Scorpion and Leaper. Unique is Crab, Beetle, Digger, Spider, Blobber, Burrower and Centipede. Island locks set the earliest island, for example Spider = 4 (`spawner_backend.lua` lines 8–17). Each island gets 3 Core, 1 Leader and 1–2 Unique (lines 1–2). Only one flyer type is allowed per island (`game.lua` line 125).
- **Developer statements.** The GDC 2019 talk "'Into the Breach' Design Postmortem" exists ([GDC Vault](https://gdcvault.com/play/1025772/-Into-the-Breach-Design)), but I found no transcript of what it says about enemies.

### The shapes and why each needs a different answer

| Shape | Counter it forces |
|---|---|
| Melee, 1 tile | Push it away, or move off the tile |
| Line, 2–3 tiles | Get out of the whole lane |
| Projectile, first hit | Block it with a body or a wall |
| Artillery arc | Walls don't help. Only moving or killing it works |
| Charge with push | Make it ram into something |
| Area, all adjacent | Don't stand next to it |
| Grapple / web | Commit to a rescue or a kill |
| Self-destruct | A race: kill it, or push it next to enemies |
| Spawner | Block the spawn tile or kill the spawner |
| Aura / shield | Change your target priority |

## 2. Duelyst

Keyword text is from `duelyst/app/localization/locales/en/modifiers.json`.

| Keyword | Text (line) | Robot role it suggests (inference) |
|---|---|---|
| Ranged | "Attack from anywhere" (77) | Artillery |
| Blast | "one straight line" (22) | Line attacker |
| Flying | "Move anywhere" (19) | Flyer |
| Provoke | "They must attack this first" (126) | Lure or decoy |
| Rush | "move and attack immediately" (120) | Banned: no attacks without a telegraph (`encounter-budget.md`) |
| Frenzy | strikes "all other nearby enemies" (52) | Area slam |
| Forcefield | "Cancel the first damage taken each turn" (49) | Shielded robot |
| Deathwatch | "whenever ANY minion dies" (34) | Reacts to a kill |
| Dying Wish | "when this minion dies" (40) | On-death drop |
| Structure | "cannot move or attack" (89) | Spawner or pylon |

- **Dying Wish appears in 39 card texts** (`cards.json`). Examples: "Deal 2 damage to nearby enemies" (1159), "Summon two 2/2 Iron Dervishes nearby" (535), "Put a random MECH minion into your action bar" (1498), and taking an opponent's artifact (573).
- **MECHAZ0R:** each of Helm, Wings, Cannon, Sword and Chassis "Progresses MECHAZ0R build by +20%" and carries one keyword (Airdrop, Ranged, Frenzy, Forcefield). The finished MECHAZ0R has "Airdrop, Forcefield, Frenzy, Ranged" (`cards.json` 1183, 1233, 1248, 1295, 1319, 1477). This is a parts-to-unit build.
- **Bosses:** D3C can't be damaged while its parts live, and each part has one keyword (`boss_battles.json` 23–35). CALIBERO 2.0 equips "a random artifact" every turn (lines 14–15).

## 3. Wesnoth

- **The mechanical race** is "animated neither by natural life nor by necromancy" (`data/core/units.cfg` lines 248–256). Its trait makes units "immune to drain, poison, and plague" (`core/macros/traits.cfg` lines 75–95). So machines ignore effects that only work on living things.
- **Machine-like units** are mostly props: the Quintain (`race=mechanical`, move 0, `HttT/units/Mechanical_Quintain.cfg` lines 5–7), walls and caravans. The Granite Golem (`HttT/units/Mechanical_Golem.cfg`) has 105 HP and move 4. Its resistance values are fire 40, pierce 40, blade 50, impact 90, cold 90 and arcane 110 (lines 46–51). In WML these are percentages of damage taken.
- **Pickups:** `PICKUPPABLE_ITEM` places an item that "can be picked up once". It lets the player "choose whether the unit stepping on it should take it", and it takes a filter for who can (`core/macros/items.cfg` lines 74–78). For example, the Storm Trident can only be taken by `race=merman` (lines 537–545).
- **Drop on death:** when Salennea dies, a Sapphire of Ice is placed at the death tile (`$x1,$y1`), and only drakes and lizards can take it (`Winds_of_Fate/scenarios/07_Harvest.cfg` lines 612–627).

## 4. "Take the enemy's weapon" designs

| Game | What happens (source) | Deterministic? | Choice? |
|---|---|---|---|
| Mega Man 11 | "Battle Robot Masters and take their powerful weapons for himself" ([Capcom search result](https://megaman.capcom.com/mm11.html)). Torch Man gives the Blazing Torch, "a weapon that diagonally fires a ball of red-hot fire" ([Capcom Unity](https://www.capcom-unity.com/2018/07/20/torch-man-brings-the-heat-to-mega-man-11/), search snippet) | Yes, one boss = one weapon | You choose the boss order |
| Horizon Zero Dawn | "Collectable components that could be shot off". Machines can be disabled "piece by piece" ([PS Blog](https://blog.playstation.com/2017/02/24/the-making-of-horizon-zero-dawns-machines/)) | Yes: break it and it's off | Which part you aim at |
| Kirby | Copy enemy powers to "steal their unique attacks". "Combine different Copy Abilities" ([Nintendo AU](https://www.nintendo.com/au/games/nintendo-switch/kirby-star-allies/)) | Yes: the enemy type decides the power | Which enemy you inhale |
| Slay the Spire | Combos are "a hallmark of our design process", with no "single dominant strategy" (Giovannetti, [Game Developer](https://www.gamedeveloper.com/disciplines/road-to-the-igf-mega-crit-games-i-slay-the-spire-i-)) | Random offer | Pick 1 of 3, or skip (see Unverified) |
| Monster Hunter | Breaking a part adds part-specific rewards (community sources only) | Random table | What you break |

## 5. Cross-source principles (inference)

1. **Each enemy type has one readable shape and one counter.** That holds in ITB's shape table and in Duelyst's one-keyword parts.
2. **The enemy's weapon becomes yours, and it's deterministic.** Mega Man, Kirby and HZD all tie the reward to the enemy you beat. Randomness only appears in which rewards you are *offered* (StS).
3. **How you kill should matter.** In HZD and MH, *where* you hit decides what you get. hello1 can make *how* a robot dies decide whether its part survives.
4. **Loot on the board creates risk.** ITB pods can be trampled. Wesnoth items wait on the death tile.
5. **Carrier filters exist (Wesnoth's race filter). hello1 can turn them into carrier variants** instead of locks.
6. **Fragile supports and spawners give the player clear priority targets** (Psion, Spider, D3C parts).
7. **Snowballing is limited by choice and slots, not by luck.** You pick one reward, carry limited slots, and the enemy budget absorbs half of each upgrade (`encounter-budget.md` §7.3).

## 6. Recommended robot roster (inference)

Power is P = HP + 2·Dmg + (move − 3) + reach (line +1, ranged or artillery +2) + 1 per push or pull tile + abilities (fly 2, shield 3, unpushable 1, extra telegraph 3) − weaknesses (1 each).

New terms I added:
- Area (3 or more tiles) +2, between Duelyst's Frenzy and Blast costs.
- A second tile in a line +1.
- Self-destruct −1.
- Spawner +3, and each spawned robot is charged against the per-turn peak when it lands.

Every nature weakness counts −1.

**Anchored rule:** anchored robots ignore pushes but *not* pulls. That gives the Dog and hook parts a job.

| # | Robot | Nature | HP | Move | Telegraph (shape) | AI quirk | P | Stage | Counter it forces |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Skitter | Light | 1 | 4, flies | Snip: 1 dmg, 1 adjacent tile (melee) | Picks the nearest animal | 1+2+1+2−1 = **5** | Early | Any hit kills it. A push flies it 3 tiles |
| 2 | Spitfire | Fuel | 2 | 3 | Flame jet: 1 dmg on 2 tiles in a line | Avoids standing next to robots (it explodes) | 2+2+1+1−1 = **5** | Early | Leave the lane, or ignite it near robots |
| 3 | Zapper | Electric | 2 | 3 | Bolt: projectile, 1 dmg to first unit hit | Robot in the way scores −2, so it repositions | 2+2+1−1 = **4** | Early | Body-block it or step aside |
| 4 | Rammer | Heavy | 4 | 2 | Charge: runs a line, 1 dmg + push 1 to first hit | Charges even if it ends next to water | 4+2−1+1+1−1 = **6** | Early | Sidestep so it hits a robot. Bait it toward water |
| 5 | Popper | Fuel | 2 | 2 | Self-destruct next turn: 2 dmg to all 4 adjacent tiles | Moves next to the most targets | 2+4−1+2−1−1 = **5** | Early | Kill it first, or push it next to robots |
| 6 | Reeler | Light | 2 | 3 | Hook: line up to 3, pulls first unit 2 tiles, 1 dmg | Scores pulling a target into another robot's telegraph | 2+2+1+2−1 = **6** | Mid | Break its line of sight or stay out of its lanes |
| 7 | Lobber | Fuel | 3 | 2 | Mortar arc over walls: 2 dmg on the center tile, push 1 on adjacent tiles | Prefers tiles behind walls (mid-game artillery) | 3+4−1+2+1−1 = **8** | Mid | Walls don't help. Reach it, or move off the tile |
| 8 | Stomper | Heavy, anchored | 4 | 2 | Slam: 1 dmg to all 4 adjacent tiles | Walks to the tile touching the most targets | 4+2−1+2+1−1 = **7** | Mid | Don't stand next to it. Pull it into water |
| 9 | Pylon | Electric | 3 | 1 | Shield beam: shields 1 robot in a line up to 3 (cancels the next damage). 0 dmg | Shields the robot with the highest score | 3−2+1+3−1 = **4** | Mid | Kill the support. A shock on it also jumps to the shielded robot |
| 10 | Lancer | Light | 2 | 4, flies | Piercing beam: 1 dmg to every unit on 3 tiles in a line (robots too) | Accepts hitting robots if it nets positive | 2+2+1+1+1+2−1 = **8** | Late | Line it up so the beam hits robots |
| 11 | Bulwark | Heavy | 4 | 2 | Maul: 2 dmg, 1 adjacent tile. Shielded (first damage each turn is cancelled) | Escorts the robot nearest the human | 4+4−1+3−1 = **9** | Late | Two hits in one turn, or push it into water (the shield doesn't stop sinking) |
| 12 | Hive | Electric, anchored | 3 | 0 | Drop pad: a Skitter lands on a marked tile next turn | Places pads next to animals | 3−1+3−1 = **4** (+5 per Skitter) | Late | Block the pad (bump 1) or kill the Hive |
| B1 | Smelter (boss) | Fuel, unpushable | 7, plus 2 arms of 2 HP | 2 | Two telegraphs: Flame Arm (line 3, 2 dmg) and Mortar Arm (arc, 2 dmg). Breaking an arm removes its telegraph | Splits its two attacks between two targets | 7+4−1+2+3+1+4−1 ≈ **19** | End of mid | Break arms to shrink the threat, or burn the core (fire deals only 1) |
| B2 | Overseer (boss) | Electric, flies | 6 | 3 | Two telegraphs: chain bolt (line; jumps 1 to each unit next to the target) and one drop pad per turn. Shields 1 robot | Shields first, then attacks | 6+4+1+3+2+3−1 = **18** (+ spawns) | Final | Kill the escorts it shields. Shock water under them |

Coverage:
- Each nature has three robots. Light: 1, 6, 10. Fuel: 2, 5, 7. Electric: 3, 9, 12. Heavy: 4, 8, 11.
- Shapes: melee (1, 11), line (2, 3, 10), charge (4), artillery (7), area (8), pull (6), self-destruct (5), support or shield (9, 11), spawner (12).
- Mid adds the anchored robot and artillery behind walls. Late adds the flyer, the shielded robot and the spawner.
- Elites are +2 HP and +1 dmg (+4 P). Following `tile-damage.md`, early robots that push deal 1.

**Battle 1 check** (human + Dog, total ≤ 22, peak ≤ 14.4):
- Turn 1: Rammer 6 + Skitter 5 = 11, ×1.25 = 13.75 ✓.
- When one dies, Zapper 4 drops in: (6+4) ×1.25 = 12.5 ✓.
- Later, Spitfire 5. Total 20 ≤ 22 ✓.

All four natures appear and only early robots are used. Popper (5) can replace Spitfire in another battle-1 layout.

### Parts each robot drops (carrier variants)

Human = the engineer (precise or ranged). Ground animal = Dog, Bear or Boar (body moves). Flyer = Eagle (acts on tiles it flies over).

| Robot → Part | Human | Ground animal | Flyer | Combo it opens |
|---|---|---|---|---|
| Skitter → Snap Claw | Swap places with an adjacent unit | Toss an adjacent unit over you to the tile behind | Carry an adjacent *ally* up to 2 tiles | Put a robot in another robot's telegraph. Rescue the human |
| Spitfire → Flame Jet | Fire on 2 tiles in a line | Fire swipe on 3 tiles in front | Fire bomb on one tile flown over | Ignite Fuel robots (they explode) |
| Zapper → Arc Gun | Shock bolt (first hit). Chains through Electric robots and water | Live wire: robots that hit it in melee get shocked | Shock one water tile flown over | Electric takes 2 from shocked water |
| Rammer → Ram Plate | Brace: unpushable this turn. Units pushed into you take +1 | Charge in a line, push 1 | Anvil: 2 dmg to the robot you land next to. Crushes Light robots | Brace for bumps. A charge into water |
| Popper → Blast Charge | Throw a charge up to 3 tiles. It blows at the end of the robot turn (1 adjacent) | Plant a mine that blows when a unit is pushed onto it (2) | Tag a robot. It explodes when it dies (Dying Wish) | Push robots onto mines. Chain deaths |
| Reeler → Winch Hook | Pull the first unit in a line 2 tiles | Reel yourself to the first obstacle in a line | Drag a hooked robot along your flight path | Drag a Heavy across water (it sinks). Pull an anchored robot |
| Lobber → Mortar Tube | Lob over walls: 1 center, push the 4 adjacent tiles outward | Recoil shot: 1 dmg in a line, then hop back 1 | Airburst on landing: push the 4 adjacent tiles outward | Scatter a cluster into water or fire |
| Stomper → Anchor Spike | Pin a robot: it can't move next turn | Stomp: 1 to all adjacent tiles. Unpushable this turn | Drop a post (wall) on a tile flown over | Keep a robot in a lane you set up. Walls for bumps |
| Pylon → Shield Emitter | Shield an ally in a line | Repulsor: when the shield breaks, push the attacker 1 | Shield every ally next to your landing tile | Absorb the one telegraph you can't dodge |
| Lancer → Beam Lens | Piercing beam on 3 tiles (hits allies too) | Mirror: line attacks that hit it turn 90° (you pick the side) | 1 dmg to every tile flown over | Redirect robot beams into robots |
| Bulwark → Riot Shield | Cover: line attacks into the tiles behind you are blocked | Shield bash: push 1 and flip the robot's telegraph direction | Drop on an ally: it ignores the next hit | Turn a telegraph back onto robots |
| Hive → Drone Bay | Launch a decoy drone (1 HP). The AI scores it like an animal (+5) | Plug a drop pad with no bump damage | Drop a decoy on a tile flown over | Lure telegraphs onto robots |
| Smelter arm → Smelter Flame / Smelter Mortar | Boss version: line 3 / arc 2 | as above | as above | Unique. Only one copy per run |

## 7. Recommended drop rules (inference)

1. **Every robot drops its weapon part, unless the way it died destroys the part.**
   - Killed by damage or a bump: the part drops.
   - Sunk in water, or a Fuel robot exploded by fire: no part. The robot drops 1 spare part instead.
   - A Popper that detonates itself: nothing.
   - Hive-spawned Skitters: nothing, so the player can't farm them.

   Show the outcome in the kill preview (a part icon or a scrap icon), so there is no hidden information. This makes "clean kill versus fast kill" a real choice (HZD, MH).
2. **The part lands on the death tile.** Any unit that ends a move there picks it up (Wesnoth).
3. **A part on the board can be lost.** A robot stepping onto it, a drop-in landing on it, or any damage on its tile destroys it (ITB pod rule). Parts still on the board at victory are collected automatically. Picking a part up early secures it. Robots never target parts, so the AI stays as it is.
4. **After the battle, keep 1 part. The rest become spare parts.** You choose from what you collected (usually 3–5), and skipping is allowed. Every part you don't keep becomes 1 spare part. This is StS's pick-1 with a deterministic pool you earned on the board.
5. **Slots cost spare parts.** The 2nd animal slot costs 2 and the 3rd costs 3, as already proposed. Because spares come from discarded parts, building slots competes with keeping good parts.
6. **Elites drop their normal part plus 1 spare part.** Bigger parts would be a stat boost, which the user dislikes.
7. **Bosses:** each broken arm drops that arm's part mid-fight (D3C, HZD). Killing the core drops a unique Boss Core worth 3 spare parts, enough for one slot.
8. **Limits on snowballing:**
   - One new part per battle.
   - Carry limits: human 4, animals 1–2.
   - Each team member may carry only one copy of a part.
   - Parts have a 2-turn cooldown or limited uses per battle.
   - Part upgrades raise the robot budget at 50% (`encounter-budget.md`).

## Unverified

- **ITB scripts** are a third-party copy, version unknown (see `enemy-ai.md`). I found no developer statement on Vek design. The GDC talk transcript wasn't available.
- **Mega Man:** I couldn't load Capcom's pages (certificate error). The quotes are search snippets. The weakness-cycle details come only from fan sources.
- **HZD heavy-weapon pickup** (for example the Thunderjaw's Disc Launcher) is reported by PC Gamer and fan wikis, not by Guerrilla. I didn't find a GDC talk on components.
- **Monster Hunter** part-break rewards come from community sources only.
- **Slay the Spire's "1 of 3, can skip"** comes from Wikipedia and guides. The developer interview covers combos, not the three-card offer.
- **Hades:** I found no first-party source on boon choices, so I didn't use it.
- **Wesnoth resistance semantics** (percentage of damage taken) are from memory, not from a doc I opened.
- **The new power terms** (area +2, second line tile +1, self-destruct −1, spawner +3) are my guesses. Robot names and stats are proposals for the solver to test.
