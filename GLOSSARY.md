# Game

A single-player tactics puzzle game where enemies telegraph their attacks and the player answers by playing cards onto a grid.

## Board

**Tile**:
One square of the grid. Attacks, summons and terrain all apply to tiles.
_Avoid_: Cell, square, hex

**Terrain**:
A tile's ground type, which gives simple fixed effects (movement cost, armor, range) or acts as a hazard.
_Avoid_: Biome

**Time of Day**:
The day/night state shared by the whole board. It moves forward on a known schedule and changes unit strength.
_Avoid_: Day/night cycle, phase

## Pieces

**General**:
The player's leader on the board. If it dies, the player loses.
_Avoid_: Hero, commander, king

**Unit**:
Any piece on the board other than the General, whether yours or the enemy's.
_Avoid_: Minion, creature, mech

**Summon**:
Putting a unit onto a tile next to one of your own units by playing a card.
_Avoid_: Spawn (used only for enemies appearing), deploy

## Cards

**Card**:
A single action the player can take by spending mana. Each card does one simple thing.
_Avoid_: Ability, skill

**Hand**:
The cards the player can play right now. The next card is visible.

**Mana**:
The resource spent to play cards. It refills every turn.
_Avoid_: Energy, AP

**Trap**:
A card played face up onto a tile that triggers when an enemy attack resolves there.

## Turn

**Telegraph**:
A visible promise of what an enemy will do this turn, which always happens exactly as shown.
_Avoid_: Intent, warning, preview

**Preview**:
The game showing the full result of a card or move before the player commits to it.
_Avoid_: Telegraph (telegraphs are the enemy's; previews are the player's)

**Resolve**:
The step where telegraphed attacks and triggers take effect, in a numbered order.

## Puzzles

**Puzzle**:
A fixed starting position, lasting one or two turns, with a goal to reach.
_Avoid_: Level, challenge, mission

**Solution**:
A sequence of player actions that reaches a puzzle's goal from a given position.
_Avoid_: Answer, walkthrough

**Solver**:
The part of the game that finds the solutions for any position.
_Avoid_: AI (the AI controls enemies)

**Hint**:
Partial help toward a solution, such as a nudge or the next move.
