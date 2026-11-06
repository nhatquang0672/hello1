# Game Design Document (draft v0.2)

Working title: TBD

Status legend: **Decided** = you chose it · **Leaning** = recommended in discussion, you haven't confirmed · **Proposal** = new idea, not discussed yet · **Open** = undecided.

## 1. Pitch

An AI has taken over its machine army and is attacking Earth. **The last human**, with a small team of **animals**, fights back using the last tools of the human era and **parts salvaged from the robots they destroy**. (Decided)

A single-player tactics puzzle game on a small grid. Robots show their attacks before they act, and you answer with a plan.

Inspired by: Into the Breach (telegraphs, clarity), Duelyst (a leader you must protect), Wesnoth (terrain), Magic: The Gathering (combos).

## 2. Priorities

1. **Simple** (Decided)
2. **Fun** (Decided)
3. **Combo building** (Decided)

Not a goal: making money. Paid solutions were dropped. (Decided)

## 3. Your team

| Member | Role | Part slots | Status |
|---|---|---|---|
| 🧍 **The human** | Leader and engineer. Salvages and fits parts. If the human dies, the run ends. | 4 (hands + backpack) | Leaning |
| 🐕 / 🐻 / 🐦 … **Animals** | Real animals, no spirits or magic. Each has one natural trait. They carry parts on a harness. | 1–2 | Leaning |

Team size: the human + 2 animals chosen at the start of a run from a small roster. (Leaning)

Example animals (Leaning):

| Animal | Natural trait | Slots |
|---|---|---|
| 🐕 Dog | Fast, drags a unit 1 tile | 2 |
| 🐻 Bear | Heavy (can't be pushed), knocks units back | 2 |
| 🐦 Crow | Flies over obstacles and water, fragile | 1 |
| 🐗 Boar | Charges in a line, pushing whatever it hits | 2 |
| 🐢 Turtle | Shell: no damage from the front, slow | 2 |

Human ability (Leaning): **hack** one robot's telegraphed attack per turn, for example to redirect it.

## 4. Starter kit: the last tools of the human era (Proposal)

Every run starts with a few everyday human tools, so you can make combos from the first battle, before you've salvaged anything. They're weaker than robot parts and get replaced over the run.

| Tool | Kind | Effect |
|---|---|---|
| **Crowbar** | Trigger | Push a unit 1 tile (impact) |
| **Flare Gun** | Trigger | Sets fuel on fire at range; can also lure a robot one tile toward the flare |
| **Jerry Can** | Setup | Pours fuel on a tile (limited uses per battle) |

Open: how many tools; whether they have limited uses; whether animals can carry them.

## 5. Robots (enemies)

- Robots **telegraph** their attacks onto tiles before acting, shown as red targeting lasers. (Leaning)
- Each robot has **one nature** with a weakness that makes sense. (Leaning)

| Nature | Examples | Weakness |
|---|---|---|
| ⛽ Fuel-powered | Tank, Flamer | Fire: explodes, damaging neighbors |
| ⚡ Electric | Drone, Taser | Shock: chains to nearby electric robots |
| 🔩 Heavy | Walker, Crusher | Pushed into water: sinks |
| 🪶 Light | Scout drone | Pushed: flies 3 tiles, not 1 |

Open: number of robots per battle; whether there's an enemy commander (a duel) or you win by surviving N turns.

## 6. Parts and salvage

- Destroyed robots drop a part on the board; a team member **moves onto the tile** to pick it up. (Leaning)
- The human fits parts between battles. (Leaning)
- A robot's nature decides what you salvage. (Leaning)

| Robot nature | Setup part | Trigger part |
|---|---|---|
| ⛽ Fuel | Oil Sprayer (coats a tile with fuel) | Igniter (sets fuel on fire) |
| ⚡ Electric | Charge Tag (marks a robot to receive shock) | Spark Arm (shoots a shock) |
| 🔩 Heavy | Anchor (pins a robot in place) | Hammer (big push and impact) |
| 🪶 Light | Magnet Tag (marks a robot to be pulled) | Magnet (pulls marked robots) |

- **The same part works differently on different carriers** (a Magnet on the Bear pulls the robot in; on the Crow, the Crow flies to the robot). (Leaning)
- **No cards:** every part's ability is always available, with a cooldown. (Leaning)

## 7. Combos

**Combo = setup + trigger × robot nature × position.** (Leaning)

Effects come **only from your parts and tools**. No hazards are placed on the map in advance, and robots don't spill things when they die. (Leaning)

Main combo sources (Leaning):
1. The order your 3 units act in
2. Turning robot attacks on robots (pushing, and the human's hack)
3. Setup + trigger parts reacting with robot natures
4. Pushing into terrain (water, walls)

Example: the Dog sprays fuel on a fuel tank robot → the human fires the Flare Gun → the tank explodes → the blast hits 2 electric drones → the shock chains between them.

## 8. Board and rules

| Rule | Status |
|---|---|
| Single-player, against an AI that telegraphs | Leaning |
| Small square grid (about 6×6 to 8×8) | Leaning |
| Telegraphs target tiles, not units | Leaning |
| No randomness in combat | Leaning |
| Simple terrain only: open ground, water, walls, buildings | Leaning |

## 9. Run structure (Open)

- Several battles per run; parts and the team carry over between battles.
- Variety between runs comes from: which animals you pick, which robots you destroy, which parts you keep.
- Open: how many battles, choosing a route, bosses.

## 10. Open questions

1. Win condition: an enemy commander (a duel) or survive N turns?
2. The starter kit (section 4): which tools, how many, with limited uses or not?
3. How many robots per battle?
4. Run length and structure
5. Title
6. Technology (a TypeScript web game was suggested but not discussed)

## 11. References

- Games: Into the Breach, Duelyst, Wesnoth, Magic: The Gathering, Fights in Tight Spaces, Slay the Spire.
- Code: `duelyst/` (CC0), `wesnoth/` (GPLv2: ideas only), `forge/` (GPLv3: ideas only).
