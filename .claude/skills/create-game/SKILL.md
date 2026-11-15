---
name: create-game
description: Create a new board game in this repository end to end — validate the name and rules, read the boardlang docs, write the .boardlang definition, rules.md and boardlang.json under games/<name>/, declare the board as a `view` block, add boardlang game tests, and check it with the gameweave CLI. Use when the user asks to create or add a game, or write a boardlang game definition.
---

# Create a New Game

## Overview

This skill creates a new game package under `games/<game_name>/` (or a module
under `modules/<name>/`), from rules to a package that validates, tests,
checks and simulates clean with the `gameweave` CLI. The checklist the result
is held to is `.claude/rules/creating-games.md` — read it first; this skill is
the process, the rule is the contract.

## Parameters

- `game_name` — The game name in kebab-case (e.g., `hex`, `go-fish`, `tic-tac-toe`)
- `game_rules` — A description of the game rules (setup, turns, movement, win/lose/draw conditions)

## Instructions

### Step 1: Validate Input

Check that the user has provided both a game name and rules. If either is
missing, ask:

```text
To create a new game, I need:
1. **Game name** — in kebab-case (e.g., `hex`, `go-fish`)
2. **Game rules** — setup, turns, valid actions, win/lose/draw conditions
```

Validate the game name:

- Must be kebab-case (lowercase letters, numbers, hyphens)
- Must not already exist as a directory in `games/` or `modules/`
- Should not already be taken on the registry by another publisher: package
  names are first come, first served, and the publish of a taken name is
  refused. If unsure, ask the user to look the name up on gameweave.com.

The license is not a choice here: every package in this repository is
`"Apache-2.0"`, or `{ "source": "Apache-2.0", "assets": "<CC id>" }` when its
art is third-party under `CC0-1.0`, `CC-BY-4.0` or `CC-BY-SA-4.0`
(`gameweave.workspace.json` rejects anything else). A game Gameweave keeps all
rights to does not belong in this repository — say so and stop.

If `games/<game_name>/` already exists, tell the user and ask whether they want
to update the existing game. **When updating an existing game:** any change to
a file the package ships (`*.boardlang` including tests, `rules.md`,
`README.md`, `assets/**`, `boardlang.json`) **requires bumping `"version"` in
`boardlang.json`**. Publishing skips an already-published version, so a change
without a bump never reaches players, and `gameweave check --changed-since`
fails it (`CLI102`).

If an issue in this repository describes the game, read it first — it is the
source of truth for rules, player counts, theme and scope.

### Step 2: Validate Rules Completeness

Check the provided rules cover these essentials:

- **Objective** — how do you win?
- **Setup** — what does the starting position look like?
- **Turn structure** — who goes when, what can they do?
- **Valid actions** — what moves/placements/choices are available?
- **Win/lose/draw conditions** — when and how does the game end?

If any are missing, ask the user for the missing parts before proceeding. For
well-known games, fill in standard rules and confirm with the user.

### Step 3: Read the Boardlang Documentation (required)

Before writing any game files, read the docs. Do **not** invent syntax or block
shapes from memory when the docs define them.

| Doc                  | Where                                               | Read when                                             |
| -------------------- | --------------------------------------------------- | ----------------------------------------------------- |
| Language syntax      | https://gameweave.com/docs/boardlang-syntax         | Always — blocks, expressions, pipes, and view blocks  |
| Block types          | https://gameweave.com/docs/block-types              | Always — `component`, `layout`, `template`, `zone`, … |
| Effect functions     | https://gameweave.com/docs/effect-functions         | Always — moves, zone ops, composition                 |
| Query functions      | https://gameweave.com/docs/query-functions          | Always — pipelines for game state                     |
| Creating games       | https://gameweave.com/docs/creating-games           | Always — the end-to-end walkthrough                   |
| Module system        | https://gameweave.com/docs/modules                  | When importing modules or shipping a reusable deck    |
| Example walkthroughs | https://gameweave.com/docs/examples                 | For genre-specific patterns                           |
| Error codes          | https://gameweave.com/docs/error-codes              | When debugging compiler diagnostics                   |
| The CLI              | `node_modules/gameweave/README.md` (after `npm ci`) | Commands, module resolution, `gameweave test`         |

The `style` vocabulary, the card template system and the test-file grammar are
not on the public docs site yet. For those, the games in this repository are
the reference — read one that already does what you need before writing it:

- **A view-driven board:** `games/tic-tac-toe/` (a `<Grid>`), `games/dominoes/`
  (a `<Plane>`), `games/hex/` (a `<Lattice>`).
- **Custom cards:** `modules/spanish-deck/` (a whole deck from `layout` +
  `template` + `from_template`), `modules/standard-deck/` (import it rather
  than copy it unless you are customizing it).
- **Game tests:** `games/tic-tac-toe/tests/rules.boardlang`,
  `games/go-fish/tests/rules.boardlang`, `games/ludo/tests/rules.boardlang`,
  `games/war/tests/rules.boardlang`, `games/gin-rummy/tests/rules.boardlang`.

**Every custom card goes through the template system** (`layout` + `template` +
`from_template` + slot `property`s). Do not invent one-off card components.

### Step 4: Boardlang-First, Views-First

**All game logic is boardlang.** Rules, scoring, phases and win conditions go in
`games/<game_name>/<game_name>.boardlang` as `query` / `effect` / `rule` /
`action` blocks. A pattern two or more games need goes in a module under
`modules/` with exported `query` / `effect` blocks. There is no engine code in
this repository and a game never needs any: compose the existing primitives
(`sequence`, `if`, `for_each`, `while`, `let`, `set_state`, zones, module
effects), and when that seems impossible, redesign the digital flow (a player
lays a meld instead of the game auto-partitioning a hand). If a mechanic truly
cannot be expressed, stop and tell the user; it is a platform gap to raise as an
issue, not something to work around inside a game.

**The board is declared, never coded.** A game's display is a `view` block
attached with `display { view = "…" }`; one shared renderer draws every
view-driven game.

| Do not                                            | Do instead                                                                                                                                                                                    |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Set `display.type` (it silently wins over `view`) | Declare `display { view = "<view_label>" }` only                                                                                                                                              |
| Call an effect function inside a `view` (BL053)   | Read state only; a press resolves against `legalMoves` by itself                                                                                                                              |
| Name a primitive outside the closed set           | Use `<Table>`, `<Row>`, `<Column>`, `<Stack>`, `<Place>`, `<Grid>`, `<Lattice>`, `<Zone>`, `<Text>`, `<Option>`, `<Input>`, `<Image>`, `<Plane>`, `<Arrow>`, `<For>`, `<Show>`, `<Children/>` |
| Write `//` comments inside a view body            | `{/* … */}` there; prose goes above the `view` block                                                                                                                                          |

Boardlang constraints that are expensive to discover late:

- **Never write an action that takes a subset of a zone.** The move generator
  enumerates every subset — a 17-card hand is 2^17 moves and hangs. Model
  multi-card plays as a repeatable single-card action into a staging zone plus a
  commit action.
- **A pass/confirm button is an action with no `param`.** It is one move with
  empty params, pressed in a view with `<Option action="pass_bid">` — see
  `action "pass_bid"` in `games/pitch/pitch.boardlang`. Never give it a
  placeholder `param "confirm"` to make it playable.
- **Boardlang has no array state.** Use zones for ordered collections.
- **An always-legal action breaks pass-and-play.** A single device only hands
  over when the viewing seat has zero legal moves, so an action every idle seat
  can always take (slap, buzz, doubt) soft-locks local play. Gate it behind a
  discrete step instead of `turn_order = "realtime"`.
- **Prefer a board that fits one screen** (`display { view = … fit = "screen" }`)
  and give a board too wide for a phone a `compact_view` — see
  `.claude/rules/creating-games.md`.

### Step 5: Build the Package

Work through these in order; each feeds the next.

1. **Design the component layout.** Decide component dimensions (portrait cards
   100×140, landscape tiles 120×60, square tokens 80×80), plan what each
   component displays, position the slots, and write the `layout` blocks.
   Layout first, art second: the layout fixes the sizes the art must fit.
2. **Assets.** Art that fits the layout slots, plus a 512×512 `assets/icon.png`
   listing icon. Every asset is your own or compatibly licensed, and none shows
   the Gameweave name, logo or diamond mark. Gameweave's AI generator
   (`npm run generate:asset`) is **maintainers only — it needs a gameweave
   monorepo checkout** (`GAMEWEAVE_MONOREPO=<path>`); otherwise draw or source
   the art, and ask the user how they want to handle it.
3. **The game definition**, `games/<game_name>/<game_name>.boardlang`:

   ```
   game "<game_name>" {
     display_name = "<Display Name>"
     description = "<Short description>"
     players = <N>
     player_1_name = "<Name>"
     turn_order = "<alternating|manual>"

     display {
       view = "board"
     }
     ...
   }
   ```

   Define, in order: layouts, assets, enums, components (referencing layouts),
   zones, mechanics, state, queries, phases/turns/actions, rules, setup. Use a
   `round` block with `end_round()` for games with repeating rounds.

4. **`rules.md`** — overview and objective, setup, turn structure, valid moves,
   special rules, win/lose/draw conditions.
5. **Map every rule to boardlang**, one mechanic at a time, using only existing
   primitives (Step 4).
6. **The `view` block** — the board, as JSX: any leading `param` blocks, then
   exactly one root element (normally `<Table width height>`, a fixed canvas
   the platform scales to fit). Factor a repeated region into a second `view`
   used as a composition. Five query functions exist only inside a view —
   `viewer()`, `seats()`, `seat_across()`, `seat_left()`, `seat_right()` — so
   one view serves every seat; a `zone` binding must be statically enumerable
   (`zone={"hand_" + seat}` inside `<For each={seats()} as="seat">`).
7. **Attach it** with `display { view = "board" }`, and nothing else in
   `display` that selects a renderer.
8. **Style it** with `style={{ … }}` and the `<Grid>` cell styles; the compiler
   rejects a property outside the vocabulary (BL067/BL068).
9. **`boardlang.json`**:

   ```json
   {
     "name": "<game_name>",
     "version": "1.0.0",
     "type": "game",
     "license": "Apache-2.0",
     "engineVersion": "<the range the games here declare today>",
     "main": "<game_name>.boardlang",
     "description": "<Short description>",
     "displayName": "<Display Name>",
     "game": { "rules": "rules.md", "players": <N> },
     "iconImage": "assets/icon.png",
     "dependencies": { "<module>": "^<version>" }
   }
   ```

   Copy `engineVersion` from a current game in this repository. List every
   module the source imports in `dependencies` (`CLI108` fails an undeclared
   import; `BL044` fails it to compile).

10. **Boardlang game tests** in `games/<game_name>/tests/rules.boardlang`,
    covering: initialization; legal actions succeed and illegal ones are
    `assert_illegal`; move execution; win/lose/draw; special rules; edge cases
    (no legal moves, full zones, after game over); and a `playout` with
    `expect_terminates = true` when the game could deadlock. Model them on the
    test files listed in Step 3.

### Step 6: Verify

Run, from the repository root:

```bash
npx gameweave validate --package <game_name>
npx gameweave test --package <game_name>
npx gameweave check --changed-since origin/main
npx gameweave simulate --game games/<game_name>/<game_name>.boardlang --modules modules
npx gameweave fmt games/<game_name>
npx gameweave lint games/<game_name> --max-warnings 0
npm run boardlang:fmt:check && npm run boardlang:lint   # what CI runs
npm run format:check
```

`simulate` must reach game over (exit `0`; exit `2` means it stopped without
one). Then run the whole gate once: `npm run validate && npm test && npm run check`.

Checking the board in a browser — chrome, sizing, the rules page, every asset —
is **maintainers only — it needs a gameweave monorepo checkout** with a local
stack. Say in the summary that it has not been checked visually unless it has.

Confirm:

1. `games/<game_name>/<game_name>.boardlang`, `rules.md` and `boardlang.json`
   exist; the manifest declares `version`, `license`, `engineVersion`, `game`
   (`rules`, `players`) and `iconImage`, with a real 512×512 `assets/icon.png`
2. Custom cards use `layout` + `template` + `from_template`
3. The game declares a `view` block and `display { view = … }`, and no
   `display.type`
4. Every command above passes

### Step 7: Present Summary

```text
## Game Created: <Display Name>

### Location
games/<game_name>/

### Files Created
- <game_name>.boardlang — Game definition, including the `view` block
- rules.md — Player-facing rules
- assets/ — Game assets, including `icon.png` (listing icon)
- boardlang.json — Package manifest (`version`, `license`, `game`, `dependencies`)
- tests/rules.boardlang — Boardlang game tests

### Board
- View block: `view "<name>"`, attached via `display { view = "<name>" }`
- Primitives used (e.g. Table, Row, Grid, Zone)
- Checked in a browser: yes / no (maintainers only)

### Modules Used
- e.g. `dice ^1.5.0` — `dice.d6`

### Testing
- validate / test / check / simulate / fmt / lint: PASS or FAIL each
- Boardlang game tests: N passing

### Next Steps
- Open a pull request. Merging to `main` publishes the new version.
```
