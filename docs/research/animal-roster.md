# Animal roster: which real animals fit the crew

Research notes for hello1. Researched 2026-11-07. Claims are cited with a URL or a local path. Anything marked **(inference)** is my own reasoning. This file builds on `game-design.md` and the other research notes and does not repeat them. Local paths are relative to `game-projects/`.

ITB script paths are from the third-party copy https://github.com/2147211690/LearnBreach (`scripts/...`), as in `hp-scale.md`.

## Key findings

1. **Every ITB starting squad is one Prime, one Brute and one Ranged or Science mech.** All 8 base squads in `scripts/squad.lua` follow this pattern (`getStartingSquad`, lines 6–29). Class decides which weapons a mech uses without a power penalty (`scripts/text.lua` line 378).
2. **Push is in every squad. A second verb makes each one different.** 4 of 8 base squads have a pull or swap (Grav Well, Attraction Pulse, Grappling Hook, Teleporter). 6 of 8 have a setup effect (smoke, rock, fire, ice, A.C.I.D.). 5 of 8 have a flyer (`scripts/pawns.lua`, `scripts/text_weapons.lua`).
3. **ITB's cheapest defensive verb is "cancel an attack" (smoke), not armor.** Rocket Artillery "creates Smoke behind the shooter". Aerial Bombs drop "an explosive smoke bomb" (`text_weapons.lua` lines 386, 503). A skunk's spray maps cleanly onto this.
4. **Two draft animals rest on behavior the sources don't support.** Mountain goats "do not fight head-to-head but rather stand side-by-side and stab at each other's flanks" ([ADW, Oreamnos](https://animaldiversity.org/accounts/Oreamnos_americanus/)). Headbutting belongs to bighorn rams, which hit "at speeds of 20 miles per hour" ([ADW, Ovis canadensis](https://animaldiversity.org/accounts/Ovis_canadensis/)). Otters eat prey in the water. ADW says nothing about drowning prey ([ADW, Lontra](https://animaldiversity.org/accounts/Lontra_canadensis/)).
5. **Deer escape. They don't fight.** ADW describes deer escaping by bounding and by entering water, with no kicking ([ADW, Odocoileus](https://animaldiversity.org/accounts/Odocoileus_virginianus/)).
6. **Three strong new verbs come from real animals.** Beavers block streams "with trees and mud" ([NWF](https://www.nwf.org/Educational-Resources/Wildlife-Guide/Mammals/American-Beaver)): terrain creation. Skunk musk reaches "up to 6.0 meters" and can cause "temporary blindness" ([ADW, Mephitis](https://animaldiversity.org/accounts/Mephitis_mephitis/)): attack cancel. Porcupine quills are "driven into the predator" on contact ([ADW, Erethizon](https://animaldiversity.org/accounts/Erethizon_dorsatum/)): counter damage.
7. **Other games tie each animal to one signature move.** Tooth and Tail sells "flamethrowing Boars", "mustard gas-lobbing Skunks" and "paratrooper-puking Owls" ([official site](https://www.toothandtailgame.com/)). Wesnoth's Boar has `specials_list=charge` (`wesnoth/data/core/units/monsters/Boar.cfg` line 65) and its Jumpcat has backstab (`Cat_Jump.cfg` line 107).
8. **Recommendation (inference):** 8 animals. Dog, Bear, Boar, Bighorn Ram, Beaver, Skunk, Eagle, Porcupine. Start with Dog, Bear and Boar. Cut Cat, Deer, Turtle and Otter. Rename Goat to Ram.

## Roles a 3-unit team needs

### Into the Breach

Base squads from `scripts/squad.lua`. Weapon text is from `text_weapons.lua`.

| Squad (internal ID) | Prime | Brute | Ranged / Science |
|---|---|---|---|
| Archive_A | Titan Fist: punch and push | Taurus Cannon: projectile push | Artemis: artillery, pushes adjacent |
| Rust_A | Jet (Brute, flies): smoke bomb | Rocket (Ranged): push + smoke | Pulse (Science): push all adjacent |
| Pinnacle_A | Burst Beam: piercing line | Ramming Engines: fly in a line and slam | Attraction Pulse: pull 1; Shield |
| Detritus_A | Electric Whip: chain damage | Grappling Hook: pull | Rock Accelerator: places a rock |
| Archive_B | Vice Fist: toss behind you | Cluster Artillery (Ranged) | Grav Well: pull |
| Rust_B | Flame Thrower: push + fire | Vulcan Artillery (Ranged): fire | Teleporter: swap |
| Pinnacle_B | Spartan Shield: flip attack direction | Janus Cannon: two directions | Cryo-Launcher: freeze |
| Detritus_B | Hydraulic Legs: leap, damage adjacent | Unstable Cannon | A.C.I.D. Projector |

The pattern is a short-range pusher, a line or reach attacker, and a utility unit (pull, swap, shield, setup).

### Duelyst keywords

From `duelyst/app/localization/locales/en/modifiers.json`:

- **Flying:** "Move anywhere on the battlefield." (mobility)
- **Provoke:** "Stops nearby enemy minions and Generals from moving. They must attack this first." (blocker)
- **Ranged:** "Attack from anywhere on the battlefield."
- **Blast:** "Attack all enemy minions and Generals in one straight line." (line)
- **Frenzy:** strike "all other nearby enemies" (area)
- **Backstab:** extra damage "when attacking from behind" (positioning)
- **Forcefield:** "Cancel the first damage taken each turn." (shield)
- **Stunned:** "lose their next action." (control)

### Wesnoth

Movetypes in `wesnoth/data/core/units.cfg` include `fly`, `swimmer`, `mountainfoot` and `woodland`. Abilities in `macros/abilities.cfg` include skirmisher (ignores zones of control), submerge, burrow and steadfast. Weapon specials in `macros/weapon_specials.cfg` include charge (double damage dealt and taken), backstab and slow. Its real animals are tuned by movetype, not by special rules. The Bear is `mountainfoot`, the Falcon is `lightfly` with 18 HP, and the Crocodile has `swamp_lurk` (`units/monsters/*.cfg`).

### Roles hello1 needs (inference)

| Role / verb | ITB | Duelyst | Wesnoth | Who covers it in hello1 |
|---|---|---|---|---|
| Short push | Titan Fist | – | – | Bear |
| Line / charge push | Taurus, Ramming Engines | Blast | charge | Boar |
| Pull | Grav Well, Grapple | – | – | Dog |
| Long knockback | – | – | – | Ram (push 2) |
| Terrain creation | Rock Accelerator | – | – | Beaver |
| Attack cancel / control | Smoke | Stunned | slow | Skunk |
| Mobility / flying | Jet, Teleporter | Flying | fly | Eagle |
| Blocker / counter | Spartan Shield | Provoke, Forcefield | steadfast | Porcupine |
| Ranged, setup, redirect | Artillery, fire | Ranged | – | **The human** (Flare Gun, Jerry Can, hack) |

The human already covers ranged, setup and redirect through the starter kit and the hack. So the animals don't need a ranged role. With only 2 animals per battle, every animal should push or move robots itself. Pure buffers waste a slot.

## Scored candidates

Each criterion is scored 0–2: (a) a real behavior mapping to one grid verb, (b) a distinct role, (c) combo hooks, (d) a readable small isometric silhouette, (e) fits a habitat event. All scores are **(inference)**.

| Animal | a | b | c | d | e | Total | Verdict |
|---|---|---|---|---|---|---|---|
| Bear | 2 | 2 | 2 | 2 | 2 | 10 | Keep |
| Bighorn Ram (was Goat) | 2 | 2 | 2 | 2 | 2 | 10 | Keep, renamed |
| Beaver | 2 | 2 | 2 | 2 | 2 | 10 | Add |
| Skunk | 2 | 2 | 2 | 2 | 2 | 10 | Add |
| Dog | 1 | 2 | 2 | 2 | 2 | 9 | Keep |
| Boar | 1 | 2 | 2 | 2 | 2 | 9 | Keep |
| Eagle | 1 | 2 | 2 | 2 | 2 | 9 | Keep |
| Porcupine | 2 | 2 | 2 | 2 | 1 | 9 | Add |
| Horse | 1 | 1 | 1 | 2 | 2 | 7 | Reserve |
| Ox | 2 | 0 | 1 | 2 | 2 | 7 | Overlaps Dog and Bear |
| Cat | 2 | 0 | 1 | 1 | 2 | 6 | Cut: overlaps Eagle, 1 damage, no displacement |
| Turtle | 1 | 1 | 1 | 1 | 2 | 6 | Cut: front-only shell needs facing rules |
| Otter | 0 | 1 | 2 | 1 | 2 | 6 | Cut: "drag under" unsupported |
| Badger | 2 | 1 | 1 | 1 | 1 | 6 | Burrow overlaps flying; no pit terrain |
| Moose | 2 | 0 | 1 | 2 | 1 | 6 | Front-hoof stomp overlaps Bear |
| Alligator | 1 | 1 | 2 | 1 | 1 | 6 | Regional; drowning not in ADW |
| Raven / Crow | 1 | 1 | 1 | 1 | 2 | 6 | No clear verb in ADW |
| Deer | 0 | 0 | 1 | 2 | 2 | 5 | Cut: escapes, doesn't kick |
| Owl | 1 | 0 | 1 | 2 | 1 | 5 | Second flyer |
| Bee swarm | 1 | 1 | 1 | 0 | 2 | 5 | Not one body; odd with 3 HP |
| Snake, Frog | 1 | 1 | 1 | 0 | 1 | 4 | Too thin or small to read |

Crab, rhino and elephant were excluded on habitat.

## Recommended roster (inference)

Damage follows `hp-scale.md`: 1 basic, 2 strong, bump 1.

| Animal | Habitat | HP | Natural move | Passive | Real behavior | Main combo |
|---|---|---|---|---|---|---|
| **Dog** | Town | 3 | **Drag:** step 1 tile away from an adjacent robot and pull it 1 tile along | Fast: +1 move | Herding dogs control livestock movement; heelers "nip at the heels" and drive animals ([Wikipedia, Herding dog](https://en.wikipedia.org/wiki/Herding_dog)). Pulling is unverified (see below) | Drag a robot into another robot's telegraphed attack, or onto a fuel tile |
| **Bear** | Forest | 4 | **Swipe:** 2 damage, push 1 | Heavy: can't be pushed | "Extremely strong", 80 to over 600 kg ([ADW, Ursus arctos](https://animaldiversity.org/accounts/Ursus_arctos/)) | Swipe a Heavy robot into water to sink it. Stand in a lane as an anchor |
| **Boar** | Forest | 3 | **Charge:** run in a straight line and push the first robot hit 1 tile, 1 damage | Thick-skulled: takes no bump damage from its own charge | Goring predators in self-defense ([Wikipedia, Wild boar](https://en.wikipedia.org/wiki/Wild_boar)); Wesnoth Boar `charge` | Charge a Fuel robot into a burning tile from across the board |
| **Bighorn Ram** | Mountain | 3 | **Headbutt:** push 2 tiles, 1 damage | Climber: can stand on ruins | Rams clash at 20 mph and climb cliffs on 2-inch ledges ([ADW](https://animaldiversity.org/accounts/Ovis_canadensis/)). NPS estimates the force at 60× what fractures a human skull ([NPS](https://www.nps.gov/jeca/learn/nature/bighorn-sheep-ovis-canadensis.htm)) | Knock a Heavy robot into water 2 tiles away, or into a second robot for a double bump |
| **Beaver** | River | 3 | **Dam:** place a log on an adjacent ground or water tile. The log acts as a wall and breaks after one bump | Swimmer: moves through water | Dams streams "with logs, branches, mud, and stones" ([ADW, Castor](https://animaldiversity.org/accounts/Castor_canadensis/)); creates "new lakes, ponds" ([NWF](https://www.nwf.org/Educational-Resources/Wildlife-Guide/Mammals/American-Beaver)) | Drop a log behind a robot, then push it in for bump damage. Block a telegraph line to shield the human. Logs burn with fuel |
| **Skunk** | Town | 3 | **Spray:** a musk cloud on the 2 tiles in a line behind it. Robots in the cloud can't attack this turn (like ITB smoke) | Warning stomp: adjacent robots telegraph away from it if they can | Sprays "up to 6.0 meters" after stomping as a warning; causes "temporary blindness" ([ADW, Mephitis](https://animaldiversity.org/accounts/Mephitis_mephitis/)) | Push a robot into the cloud to cancel it. Cancel one robot so another walks into its own ally's fire |
| **Eagle** | Mountain | 3 | **Snatch:** lift an adjacent Light robot and drop it up to 2 tiles away. On other robots: 1-damage dive | Flying: crosses water and walls | Carries "up to 8 pounds"; dives at "up to 200 mph" ([ADW, Aquila](https://animaldiversity.org/accounts/Aquila_chrysaetos/)) | Drop a Light drone into water, onto a fuel tile, or next to Electric robots before a shock |
| **Porcupine** | Forest | 3 | **Back into:** 1 damage to an adjacent robot, no push | Quills: any robot that attacks it in melee or is bumped into it takes 1 extra | Quills work "when they are driven into the predator" ([ADW, Erethizon](https://animaldiversity.org/accounts/Erethizon_dorsatum/)) | Stand where robots will be pushed: every bump deals 3 total. Stand in a melee telegraph to punish it |

Habitat coverage: town 2, forest 3, river 1, mountain 2. River is thin. Add a second river animal later if events need one **(inference)**.

Notes on the passives:
- Skunk's "warning stomp" is a soft Provoke in reverse. If it makes the AI unclear, drop it and give Skunk Fast instead.
- Bear is the one sturdy animal at 4 HP, as `hp-scale.md` recommends.
- Spray has no cooldown, like ITB's Rocket Mech smoke. It cancels one or two robots per turn, so test it first for balance.

## Recommended starting 3 (inference)

**Dog, Bear, Boar.**

- It mirrors ITB's Archive_A squad: short push (Bear), line push (Boar) and a utility mover (Dog, pull).
- All three use only push and pull. The player learns bump, water and robot natures before any new rule.
- Each later animal adds exactly one new verb: long knockback (Ram), terrain (Beaver), attack cancel (Skunk), flight (Eagle), counter damage (Porcupine). Each habitat event then teaches one rule.
- Town and forest fit an early run. River and mountain events come later.

## Unverified

- **Dog pulling.** The herding source supports driving and heading, not dragging. Dogs tugging and gripping is common knowledge, but I didn't find an authoritative source. "Herd" (turn a robot's telegraph) is a sourced alternative.
- **Boar charge.** No ADW or agency page describes charging. The evidence is Wikipedia's goring reports and news outlets ([search summary](https://outdoorhub.com/?p=400586)). Britannica returned 403.
- **Porcupine backing into predators and tail swings.** Found on Wikipedia, encyclopedia.com and a zoo page, not on ADW. The quills passive is ADW-backed. The natural move is not.
- **Eagles knocking goats off cliffs.** Only filmed incidents (BBC Frozen Planet II, as reported by the Daily Express). I found no study. The roster doesn't depend on it.
- **Alligator death roll and drowning.** Not on the ADW page. Alligators do use "man-made water structures ... in urban and suburban areas" ([ADW](https://animaldiversity.org/accounts/Alligator_mississippiensis/)).
- **Moose front-hoof kicks** are well supported ([ADF&G](https://www.adfg.alaska.gov/index.cfm?adfg=wildlifenews.view_article&articles_id=939)), but Moose overlaps Bear.
- **ITB squad display names.** The scripts use internal IDs (Archive_A and so on). Matching them to names like "Rift Walkers" is from memory.
- **The ITB scripts are a third-party copy.** See `enemy-ai.md` for why it looks genuine.
- **Other games.** Root's Leder Games page returned 429. Armello's official site doesn't describe clan strengths. I didn't check Wildfrost or Ratropolis first-party pages.
- **Skunk musk flammability** is unchecked. Don't build a fire combo on it.
