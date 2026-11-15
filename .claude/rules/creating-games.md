# Creating a New Game

When adding a game (or a module) to this repository, follow this structure and
checklist. Every command below is the `gameweave` CLI pinned in `package.json`;
run it through `npx gameweave` or the npm scripts.

The language is documented at **gameweave.com/docs**:
[syntax](https://gameweave.com/docs/boardlang-syntax),
[block types](https://gameweave.com/docs/block-types),
[effect functions](https://gameweave.com/docs/effect-functions),
[query functions](https://gameweave.com/docs/query-functions),
[modules](https://gameweave.com/docs/modules),
[error codes](https://gameweave.com/docs/error-codes),
[creating games](https://gameweave.com/docs/creating-games) and
[examples](https://gameweave.com/docs/examples). The CLI's own README (on
[npm](https://www.npmjs.com/package/gameweave), and at
`node_modules/gameweave/README.md` after `npm ci`) documents every command,
module resolution and the `gameweave test` runner. Where neither covers a
feature (the `style` vocabulary, card templates, the test-file grammar), the
games in this repository are the reference: find one that already does it.

## Directory Structure

Each game lives in its own directory under `games/`, each module under
`modules/`:

```
games/<game-name>/
├── boardlang.json           # Package manifest (required; version, license, dependencies)
├── <game-name>.boardlang    # Game definition (required)
├── rules.md                 # Game rules in markdown (required)
├── README.md                # Optional; required for attribution of third-party art
├── tests/                   # Boardlang game tests, run by `gameweave test`
│   └── rules.boardlang
└── assets/                  # Game assets directory (optional)
    ├── header.svg           # Optional list/header image (see headerImage in boardlang.json)
    ├── icon.png             # Square listing icon (see iconImage)
    ├── <piece>-<player>.svg # Piece images per player
    └── ...
```

A module has the same shape under `modules/<module-name>/` with
`"type": "module"` in its manifest. A module with tests ships a
`tests/harness.boardlang` game to run them against.

Use kebab-case for every directory name; the directory name is the package
`name` in `boardlang.json`.

## Required Files

### 1. Boardlang File (`<game-name>.boardlang`)

The main game definition. Must contain a top-level `game` block with:

- `description` - Short description string
- `players` - Number of players (number)
- `player_1_name`, `player_2_name` - Player display names
- `turn_order` - Either `"alternating"` or `"manual"`

Interaction is **derived**, not declared: a game defines parameterized `action`
blocks (`param` types `cell` / `piece` / `zone_piece`) and the platform resolves
a press against `legalMoves`. There is no `interaction_mode` attribute.

### 2. Rules File (`rules.md`)

A markdown file describing the game rules for players. Should include:

- Game overview and objective
- Player information (who goes first, how turns work)
- How pieces move or how the game is played
- Special rules or moves
- Win/draw conditions
- Initial setup description

### 3. Assets (`assets/` directory)

Piece images plus listing artwork. **Every asset must be your own work or carry
a license compatible with this repository** (see "Package license" below), and
**no asset may show the Gameweave name, logo or diamond mark.**

**Listing icon:** a square `assets/icon.png` (512×512 PNG), named in
`boardlang.json` as `"iconImage": "assets/icon.png"`. The catalog falls back to
a letter tile when `iconImage` is omitted; a new game should not ship that
fallback.

Gameweave's AI asset generator (`npm run generate:asset`) is **maintainers
only — it needs a gameweave monorepo checkout**: run it from that checkout,
with `GAMEWEAVE_MONOREPO=<path>` set for any script here that calls it. Anyone
else draws or sources the icon and art by hand.

**Piece images:** SVG (or PNG). Convention:

- Named as `<piece-name>-<color-or-player>.svg` (e.g., `pawn-white.svg`, `pawn-black.svg`)
- Or `<piece-name>.svg` if not player-specific (e.g., `stone.svg`)
- Referenced in the boardlang file via `asset` blocks with `src = "assets/<filename>"`

### 4. Board Display (a `view` block)

A new game declares its board **in boardlang**, as a `view` block attached with a
`display` block:

```
display {
  view = "board"
}

view "board" {
  <Table width={582} height={408}>
    <Row gap={15} align="center">
      <Zone zone="stash_1" orientation="stack" style={{ width = 52 }} />
      <Grid rows={6} columns={7} cell_shape="circle" background="#1e3a8a" />
      <Zone zone="stash_2" orientation="stack" style={{ width = 52 }} />
    </Row>
  </Table>
}
```

One shared renderer draws every view-driven game, so a game needs nothing
outside its own package to be drawn. Appearance comes from `style` attributes
drawn from a closed vocabulary; the compiler rejects a property outside it
(BL067/BL068).

Constraints worth knowing before designing a board:

- **Do not also set `display.type`.** It wins over `display.view` (the
  mid-migration escape hatch for older games), so a stray `type` silently
  bypasses the view.
- **A view cannot change state.** The whole effect registry is banned inside a
  `view` block (BL053), and there are no event handlers — a press resolves
  against `legalMoves`, exactly as for `action` blocks above.
- **The primitive set is closed**: `<Table>`, `<Row>`, `<Column>`, `<Stack>`,
  `<Place>`, `<Grid>`, `<Lattice>`, `<Zone>`, `<Text>`, `<Option>`, `<Input>`,
  `<Image>`, `<Plane>`, `<Arrow>` draw, and `<For>`, `<Show>`, `<Children/>`
  are interpreted. Square-lattice edges (connect-the-dots) and triangular
  lattices are not expressible yet, so confirm the board fits before
  designing it. A board the primitives cannot express is a platform gap: open
  an issue describing it rather than working around it.
- **A hex board is a `<Lattice>`.** `projection="hex"` draws an axial board
  (Hex, Chinese Checkers); `projection="hex_quarter"` gives every tile,
  corner and edge its own cell, so a game that builds on corners and edges
  presses them like any cell. Write the board's tiles as a `sites` block and
  name it in `site_map`; take every relation (a tile's corners, an edge's
  ends) from the `hex-lattice` module rather than a table. Where kinds crowd
  on a phone, make only one kind legal at a time. See `games/hex/` and
  `games/chinese-checkers/`.
- **A table with no fixed shape is a `<Plane>`, not a `<Grid>`.** A line of
  dominoes runs whichever way the tiles fall, so its board must be sized for
  the longest line the set can make — and a `<Grid>` of all of it shrinks the
  tiles to specks. `<Plane>` draws the pieces on the board framed to wherever
  they are (a two-cell tile once, turned the way it was laid), and a
  `<Place row column>` inside it stands on a cell: that is how a view marks
  the open ends a player presses. See `games/dominoes/`.
- **An `<Arrow>` points at boxes, never at coordinates.** Give each box an
  `anchor` id and write `<Arrow from="orb" to={"enemy_" + n} />`; the platform
  routes it round the other boxes and paints it above the whole board. It is
  hidden from screen readers and never takes a press, so state the relation in
  words too, and hide arrows with `<Show>` while a takeover panel is up.
- **Comments inside a view body are `{/* … */}`, never `//`.** The lexer is in
  JSX mode in there, so a `//` line is ordinary TEXT — and one mentioning a tag
  name is lexed as a real element, which fails the compile with a tag mismatch
  a long way from the comment. Prose belongs above the `view` block, where HCL
  comments apply.
- **A `zone` binding must be statically enumerable.** `zone={"hand_" + seat}`
  compiles when `seat` is a `<For>` binding over `seats()`, because the
  compiler can check every value it takes. `zone={"hand_" + viewer()}` does
  not: a function call has no finite domain. Draw the viewer's own zone from
  the `seat == viewer()` arm of the same `<For>` instead.

**A board too wide for a phone needs a compact view.** `<Table>` scales the
whole canvas to fit, so a 1100-wide board on a 390px phone draws a 48-unit
control at 16px, far under the 44px touch floor. Author a second, narrower
`view` and name it: `display { view = "board" compact_view = "board_compact" }`.
The platform draws it whenever the board is under 640px wide, and `view` at
every other width. Size it for a finger (a table about 460 units wide draws a
56-unit control at about 45px), stack it vertically unless the game fits the
screen (below), and reuse the main view's compositions. `compact_view` without
`view` is a compile error (BL003).

**A board that must be seen whole can fit the screen.** By default the canvas
fits the board column's width and the page scrolls past a taller board.
`display { view = "table" fit = "screen" }` gives the board the visible screen
instead: the page never scrolls, and `<Table>` scales its canvas by whichever
axis is tighter. Give each view a canvas with a fixed width AND height that suit
the screens it is played on — a canvas far taller than the screen now draws
small instead of scrolling. `fit` without `view` is BL003; any value but
`width` or `screen` is BL029.

**Type has two faces of its own, and a game may ship a third.** A `<Text>`
sets `typeface = "display"` for the theme's serif (plaques, titles, large
numerals) or `"body"` for the app's face. A game that needs its own face ships
it in the package as WOFF2 under 1 MiB, declares
`asset "x" { type = "font" src = "assets/x.woff2" fallback = "display" }`, and
names it `typeface = "x"`. The board waits for it before its first paint, so
nothing swaps mid-game; the registry parses it at publish and refuses one
that does not parse. Never name a raw font family, and never a URL: a font
loads from the package's own bytes or not at all. The font's license must allow
redistribution under this repository's terms. See `games/nim/`.

**A game's prompt and its one primary belong in the platform dock.** Drawn
inside `<Table>`, a prompt sentence and a Confirm button scale with the board,
so a phone shrinks them with everything else. Name a dock view instead:
`display { view = "table" dock = "dock" }`, where `view "dock"` has one root,
`<DockBar prompt={…} tone={…}>`, holding `<Option>` buttons and any spendable
resource as `<DockChits label count spend>` (never a number on the table). The platform
draws it below the board, outside the scaled canvas, at full size: every option
is the app's button (`primary` unless it names `variant="ghost"` for a
secondary such as Cancel), at least 48px, and hidden while no legal move offers
it. `tone = "act"` says the decision is the viewer's; `"watch"`, the default,
that someone else is deciding. The bar takes no `style` and neither do its
options (BL133); a `<DockBar>` anywhere but a view's root is BL131.

**A board drawn as art under a transparent `<Grid>` should explain itself.**
The grid is the hit target over every drawn feature, so a map's doors, cases
and panels cannot carry a `hint` of their own. Give the grid
`cell_hint={cell => map_hint(cell)}`: a query returning one sentence, or
`{ title = "…", body = [ … ] }`, for each feature, and `""` for bare floor so
nothing pops up there. A player hovers a square, or presses and holds it on
touch, to read it.

Some older games in this repository select a built-in renderer with
`display.type` instead of declaring a `view`. That path is legacy — never copy
it into a new game.

### 5. Card layouts, for a game with a deck

A card drawn inside a `<Zone>` is **60px wide**, whatever the zone asks for.
That is right for the table — a hand of thirteen has to fit in a seat — and it
is far too small to read. The platform's answer is the **card inspector**:
hovering a card on desktop, or pressing and holding it on touch, grows it into
a floating panel showing the card large beside its display name, its
`description`, and every value its text slots print. Every view-driven game has
it and no game wires it up.

So a card layout has **two** readings, and it owes both:

- **the table thumbnail** — at 60px only the art and the silhouette survive, so
  a card must be identifiable from those alone;
- **the full-size read** — a real title, the art, the cost and the rules text,
  positioned and sized as they would be on a printed card.

A layout designed only for the thumbnail is unreadable at exactly the moment a
player finally asks what the card does.

Custom cards go through the template system — `layout` + `template` +
`from_template` with slot `property`s — never one-off card components. See
`modules/spanish-deck/` for a whole deck built from templates, and
`modules/standard-deck/` for the French-suited pack (import it rather than copy
it unless you are customizing it).

Two things the inspector needs the game to supply:

- **a `description` on each card component** — one or two sentences of lore, in
  the game's own vocabulary. It is the only line in the panel that is not
  already printed on the card, and a component without one simply shows no lore.
- **text slots holding sentences, not identifiers.** The panel prints slot
  values verbatim and unlabelled, in the order the layout declares them, and
  drops a `title` slot that only restates the display name. A slot holding a
  bare number reads as a bare number.

A `<Zone>` can also state `piece_size` to draw its occupants larger or smaller
than the 60px default, which is how a feature row and a crowded hand get
different sizes without two layouts.

### 6. Dice, and what a player can read off one

A die in a `<Zone>` is inspectable the same way a card is — hover, or press and
hold. A die no card binds has no card template, so the panel shows something
else: the component's **`display_name`**, its **`description`**, and a strip of
**every face it can land on**, with the one it is currently showing ringed.

That strip keeps duplicates. A roll is uniform over `face` blocks, not over
values, so a `[1, 1, 2, 5, 5, 6]` die shows six slots and a player can see at a
glance that a 1 is twice as likely as a 2. It is also why face ORDER and
repetition in the source are worth getting right: they are visible. Faces are
drawn by VALUE — every face valued `5` draws the first such face's art — so
faces that share a value should look alike.

The `description` is load-bearing in a way a card's is not. A card prints its
own rules through its slots; a die prints nothing at all, so its `description`
is the **only** place the game can say what pressing it would do. Write it to
cover the whole face set rather than one face:

```
component "obol_hound" {
  type = "die"
  display_name = "Oathbound Hound"
  description = "Attack 1, survives 2 wounds. Three of six faces are Energy: press to bank energy. Three are Ability: press to summon the Hound for 3 energy, or 1 on the boosted face."
  ...
}
```

A die with no `description` shows its name and its faces and nothing else,
which tells a new player which faces exist and nothing about what they mean.

**Compose a die's faces from a face layout** rather than drawing an image per
face. A `face` with `layout = "<layout>"` fills that layout's slots from its own
attributes, so one layout (a coloured tile, the face's mark) serves every face
of every die. Every image is another file each player downloads and the
registry moderates, and thirty custom six-faced dice drawn face by face would
need 180 of them:

```
layout "obol_face" {
  width = 64
  height = 64
  background_color = "#3a2f5b"
  slot "mark" { type = "text" x = 0 y = 0 width = 64 height = 64 align = "center" }
}

face "ability_a" { value = 5 display_name = "Ability" layout = "obol_face" mark = "AB" }
```

**A die that stands for a card should be bound from that card.** In a
dice-and-card game the card is where the rules are printed, so give the card
`die = "<die>"` and a `die_faces` slot in its layout:

```
component "hound_card" {
  from_template = "kit_card_tpl" // its layout has slot "faces" { type = "die_faces" ... }
  die = "obol_hound"
}
```

- **The card prints the die's faces** through the `die_faces` slot: one cell per
  declared face, duplicates kept, laid out by `orientation` (`row` or
  `column`), `gap` and `align`.
- **Holding the rolled die opens the card**, drawn large, with every face in its
  strip that carries the rolled value ringed. The name and lore are the card's,
  and the die's `description` leads the lines, so write it as the one sentence
  about this die. A card whose layout prints no strip gets the die's faces
  under it instead.
- **Mark a badge numeral `inspect = false`** — a cost, a level — so it stays on
  the card and out of the panel's lines.
- **One card per die** (BL039); the die must declare faces (BL037) and the card
  must have a layout (BL038). Bind an imported die by its qualified label,
  `die = "dice.d6"`. The binding is presentation only: the rules still link a
  card to its dice however the game likes.

## Importing reusable modules

Every module a package imports must be listed in its `boardlang.json`
`dependencies`, with a semver range:

```json
"dependencies": { "trick-resolution": "^1.2.0" }
```

Dependencies come from the manifest, not from where files sit: a module in this
repository's `modules/` resolves from here (and must satisfy the range — a
mismatch is an error, `CLI012`, not a fall-through to the registry); anything
else resolves from the registry. An import the manifest does not declare fails
to compile (`BL044`), and a declared dependency nothing can supply fails with
`CLI013`. A package licensed `Apache-2.0` may never depend on an `UNLICENSED`
one (`CLI111`).

Prefer named imports for the module functions your game actually calls
unqualified, rather than a bare namespace import:

```
import { trick_winner } from "trick-resolution"
```

A namespace import (`import "trick_resolution" from "trick-resolution"`) only
makes **qualified** access work (`trick_resolution.trick_winner(...)`) — it
does **not** make a bare `trick_winner(...)` resolve, even though the module
exports it. Mixing both in one statement (`import "td", { trick_winner } from
"trick-resolution"`) is fine when a game needs some names qualified and others
bare. There is no bare-name fallback through an imported namespace, and no
transitive visibility: a module your game does not itself import is invisible
even if another module your game imports pulls it in. See
[modules](https://gameweave.com/docs/modules) for the full grammar.

## Formatting and linting

Keep every `.boardlang` file in the formatter's shape and lint-clean:

```bash
npx gameweave fmt games/<name>            # rewrite into the canonical shape
npx gameweave lint games/<name> --fix     # apply the safe fixes; read what is left
npx gameweave lint games/<name> --max-warnings 0
```

What lint leaves is yours to fix by hand. Give every `query` and `effect` a
one-line `description` (BL906), and keep attributes above a block's nested
blocks (BL904). A rule that fires on something deliberate takes a disable
comment with a reason:
`// boardlang-lint-disable-next-line no-unused-import -- <what the import provides>`.

CI gates both over every `.boardlang` file in the repository
(`npm run boardlang:fmt:check`, `npm run boardlang:lint`, which allows no
warnings). `npm run boardlang:fmt` and `npm run boardlang:lint:fix` fix the
whole repository at once. Prettier formats the markdown and JSON here
(`npm run format`) and never touches `.boardlang`.

## Package version — bump on every content change

Each game and module ships a `boardlang.json` with a semver `"version"`.
Publishing is **idempotent per version**: an already-published `name@version`
is **skipped**, so asset or source edits never reach players unless the
version changes. `gameweave check --changed-since <base>` fails such a change
(`CLI102`), and CI runs it on every pull request.

**Whenever you change any file that ships in the package, bump `"version"` in
that package's `boardlang.json`.** That includes:

- every `.boardlang` file, tests included
- `rules.md` / `README.md`
- Any file under `assets/` (piece art, `header.svg`, sounds, etc.)
- `boardlang.json` fields that affect the published package (e.g. `headerImage`, `iconImage`,
  `displayName`, `description`, `dependencies`, `license`)

Files a package does not ship (an `art/` source folder, a generator script)
need no bump.

Use a **patch** bump for asset/copy/header fixes (e.g. `1.0.0` → `1.0.1`), and
minor/major when gameplay or public API changes. If you change a module under
`modules/`, bump **that** module's version too, and update dependent games'
`dependencies` ranges when needed.

## Package license — required on every boardlang.json

Everything in this repository is open source under Apache-2.0, and
`gameweave.workspace.json` admits nothing else (`CLI101`). A manifest declares
one of:

- `"license": "Apache-2.0"` — the default, for source and assets alike;
- `"license": { "source": "Apache-2.0", "assets": "<id>" }` — when the art is
  third-party under `CC0-1.0`, `CC-BY-4.0` or `CC-BY-SA-4.0`. The package
  README then carries an attribution section (author, source, license, what
  was changed), and the item is listed under "Third-party materials" in the
  repository's `NOTICE`.

A game Gameweave keeps all rights to (`UNLICENSED`) does not belong in this
repository. A missing or unknown value fails `gameweave validate`, `pack` and
`publish`, and the registry refuses it.

## Publishing

Games are not bundled with the Gameweave app: it lists and loads them from the
registry at runtime. **Merging to `main` publishes**: after CI passes on
`main`, the Publish workflow submits every package version the registry does
not have yet, modules before the games that use them. There is nothing to
register and no app code to change — a view-driven game is playable as soon
as its version is published.

Assets travel inside the package archive, and relative
`src = "assets/<filename>"` paths resolve against the package's own files.

## Verification

After adding or changing a game:

1. `npm run validate` — every package compiles (or
   `npx gameweave validate --package <name>` for one).
2. `npm test` — every package's boardlang tests pass (or
   `npx gameweave test --package <name>`). Game rule tests live in
   `games/<name>/tests/**/*.boardlang`.
3. `npm run check` — licenses, dependencies and manifests are publishable. Add
   `--changed-since origin/main` to also check version bumps, as CI does.
4. `npx gameweave simulate --game games/<name>/<name>.boardlang --modules modules`
   reaches game over (exit `0`; exit `2` means it stopped without one).
5. `npx gameweave fmt` and `npx gameweave lint` are clean for the package.
6. `npm run format:check` passes.

Checking the board in a browser before publishing — that it renders from its
`view` block, sizes correctly on a phone, and that the rules page and every
asset load — needs a local Gameweave stack and is **maintainers only — it needs
a gameweave monorepo checkout**. Everyone else sees the board once the version
is published; reviewers check it there.
