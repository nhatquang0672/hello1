# Game Design Document (draft v0.1)

Working title: TBD. Terms are defined in [GLOSSARY.md](../GLOSSARY.md).

Status legend: **Decided** = you chose it · **Leaning** = recommended, you haven't confirmed · **Open** = not discussed yet.

## 1. Pitch

A single-player tactics puzzle game. Enemies **telegraph** their attacks. You answer by **playing cards** onto a small grid where **terrain** decides what's possible. If your **General** falls, you lose.

> Hook: you can only summon next to your own units, so to stop a threat your General has to move close to danger.

Inspired by: Into the Breach (telegraphs, clarity), Duelyst (General, summoning, cards), Wesnoth (terrain, time of day), Magic: The Gathering (keywords, combos).

## 2. Design pillars

1. **Clarity first.** Show the player everything. No randomness in combat. A mechanic that makes the board harder to read has to earn its place. (Leaning)
2. **Cards on the board.** Where you play a card matters as much as which card it is. (Leaning)
3. **Combos come from the board, not from card text.** Cards are one simple verb each. The depth comes from position, order, terrain and telegraphs. (Leaning)

## 3. Core rules

| Rule | Status |
|---|---|
| Single-player only, against an AI that telegraphs | Leaning |
| Square grid (about 6×6 to 8×8) | Leaning |
| No randomness in combat; terrain gives flat bonuses, never dodge chances | Leaning (required if you sell solutions) |
| Telegraphs target **tiles**, not units | Leaning |
| You start with only your General on the board and summon units from cards next to your own units | Leaning |
| Your General can fight | Leaning |
| Mana goes up automatically each turn (Duelyst style, no lands) | Leaning (fixed mana for one-turn puzzles) |
| Hand: no random draws mid-puzzle; the next card is visible | Leaning |
| A cap on summoned units on the board (about 3 besides the General) | Leaning |
| Free undo during your turn | Leaning |
| A preview of the full result before you commit (damage, pushes, chain reactions) | Leaning |

## 4. Turn loop

```
1. Enemy intent   enemies move, then telegraph their attacks onto tiles
2. Your turn      you get mana; your General moves and acts; you play cards; your units act
3. Resolve        telegraphed attacks land, in a visible numbered order
4. World tick     day/night moves forward; terrain effects apply
```

## 5. Win and lose

- **Lose:** your General dies. (Leaning)
- **Win:** Open, but this decides whether the game feels too close to Into the Breach:
  - (a) Survive N turns. This is Into the Breach's model, and the risk is the game reads as Into the Breach with cards.
  - (b) **Recommended:** a duel. The enemy has a General that also summons and telegraphs its summons. You win by killing it.

## 6. Combos (chosen for clarity)

Start with:
1. **Turning their attacks on them:** push an enemy onto a tile another enemy is about to hit.
2. **Sequencing:** the same cards give different results in a different order.
3. **Terrain changes:** forest burns, water freezes into walkable ice, ice melts.

Later: status + element combos (around 5 statuses at most), summoning as setup (summoning to reach further), and **trap cards** (played face up, they trigger while attacks resolve).

Clarity rules: around 8 keywords at most, no hidden triggers, effects resolve in a visible numbered order, a full preview, and free undo.

## 7. Terrain and time

- A few terrain types with simple flat effects (for example, forest gives +1 armor, high ground gives +1 range), plus impassable tiles and hazards (water, chasms).
- Pushing units into hazards is how terrain gets dramatic.
- A **day/night cycle** that you can see coming: a telegraph for the whole map that changes unit strength.
- Not using: Wesnoth's per-unit terrain tables or dodge chances.

## 8. Content structure

| Mode | Description | Status |
|---|---|---|
| Hand-made levels | Tutorial and teaching the rules; free | Leaning |
| Generated puzzles | One unique puzzle per player, from a seed, checked by the solver | Leaning |
| Daily puzzle | One short turn everyone gets the same; quick to pick up | Leaning |
| Battles / runs | Multi-turn battles, possibly in a roguelite run | Open (later) |

**Puzzle = 1 or 2 turns** (Leaning). That keeps the solver exact. Longer battles get reviewed one turn at a time.

**What makes a generated puzzle good enough to keep:** no single card solves it; every solution uses at least 2 cards interacting; there are few solutions.

What counts as solved: Open. (a) a perfect win, (b) a score, as in Into the Breach (recommended for battles), or (c) both.

## 9. Monetization

- **Decided:** players can pay for the **solution** to a puzzle.
- **Leaning:** the chess.com model. Free daily puzzle and a limited number of generated puzzles; a subscription unlocks unlimited puzzles, the archive and **solver review** of your play; single solutions are also for sale.
- **Leaning:** several levels of help, each based on the player's current position: a free nudge, a paid next move, a paid full solution.
- Solutions must be computed on a **server** so the game can't be cracked.
- Open: which audience comes first, hardcore tactics players or casual daily-puzzle players.

## 10. Technical direction

- TypeScript web game. Game rules are plain logic code, separate from the drawing code. (Leaning)
- Every move is an action object (the Command pattern). One system powers the preview, undo, replays, the solver and solution review.
- The solver runs on a server; you need a server for accounts, payments and solutions anyway.

## 11. Scope: cut or delay

| Idea | Verdict |
|---|---|
| Player vs player | Cut (breaks the telegraphs) |
| Wesnoth's per-unit terrain tables and dodge chances | Cut |
| Deck building | Delay (start with a fixed deck per General) |
| Many Generals or factions | Delay (start with 1) |
| Multi-island roguelite structure | Delay |

**First milestone:** one hand-made battle on a 6×6 board, with 1 General, 4 fixed cards, 3 terrain types and enemies that telegraph. Prove it's fun before building anything else.

## 12. Risks

1. **Scope.** Into the Breach took 2 experienced developers 4 years.
2. **Too close to Into the Breach.** Needs the duel and summoning hook to stand apart.
3. **Fights in Tight Spaces** (cards, a grid and telegraphs) is the closest competitor. Play it.
4. **Audience mismatch.** Subscription buyers are casual players; the game's depth suits hardcore players.
5. **The generator is the hardest technical piece:** puzzles that are interesting, not just winnable.
6. **Licenses.** Duelyst is CC0, so its code can be reused (check the assets). Wesnoth (GPLv2) and Forge (GPLv3) are for ideas only.

## 13. Open questions

1. Win condition: survive N turns, or a duel with the enemy General?
2. Which value comes first: clarity, discovery, attachment, or cards on the board?
3. Should trap cards be in the first version?
4. What counts as solving a puzzle: a perfect win, a score, or both?
5. Which audience comes first?
6. Rising or fixed mana (depends on the puzzle length)?

## 14. References

- Games: Into the Breach, Duelyst, Fights in Tight Spaces, Slay the Spire, Hoplite, Wesnoth, Possibility Storm puzzles.
- Code: `duelyst/app/sdk/` (`actions/`, `challenges/`, `intentType.coffee`), `duelyst/server/ai/`; `wesnoth/data/core/terrain.cfg`, `schedules.cfg`; `forge/forge-gui/res/puzzle/*.pzl`.
- Talks and articles: GDC "Into the Breach Design Postmortem"; Red Blob Games; Game Programming Patterns, "Command"; Mark Rosewater, "Making Magic".
