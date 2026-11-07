# Gameweave Games

The open catalog of board and card games for [Gameweave](https://gameweave.com),
written in **boardlang**, the language Gameweave uses to define a game as a
structured, machine-readable spec. Each game here is published to the Gameweave
registry, where anyone can play it in the browser, and each module is a
reusable piece (a deck, a set of dice, a scoring rule) that games import.

Boardlang is compiled, tested and published with the
[`gameweave` command-line tool](https://www.npmjs.com/package/gameweave). Its
README documents every command; the language itself is documented at
[gameweave.com/docs](https://gameweave.com/docs/boardlang-syntax).

> **The catalog has not landed yet.** This repository currently holds the
> tooling only: the workspace configuration, CI and the publish workflow. The
> games and modules arrive in one import, together with `CONTRIBUTING.md`,
> `NOTICE` and `TRADEMARKS.md`.

## Layout

```
games/<name>/      a game: <name>.boardlang, rules.md, boardlang.json, tests/, assets/
modules/<name>/    a module other packages import: decks, dice, shared queries
```

Every directory under `games/` and `modules/` is one package with its own
`boardlang.json` (name, `version`, `license`, `dependencies`). The packages
together form a gameweave workspace, declared by `gameweave.workspace.json`, so
the CLI can check them all at once and in dependency order: a module before the
games that use it. A game that depends on a module in this repo always gets this
repo's copy.

## Quick start

You need Node.js 24 (the exact version is in `.node-version`).

```bash
npm ci
npm run validate     # compile every package
npm test             # run every package's boardlang tests (tests/**/*.boardlang)
npm run check        # licenses, version bumps, dependencies: ready to publish?
```

To play a game through to the end with automated moves:

```bash
npx gameweave simulate --game games/chess/chess.boardlang --modules modules
```

Each command also takes `--package <names>` (a comma-separated list) or
`--changed-since <ref>` (what changed since a branch, plus everything that
depends on it) instead of checking every package.

## Make a new game

- The [creating games guide](https://gameweave.com/docs/creating-games) walks
  through a game from an empty folder.
- The games already here are the best examples there are. Find one with a
  similar shape (a grid, a hand of cards, dice) and start from it.
- `.claude/rules/creating-games.md` is the checklist this repo holds every game
  to, and `.claude/skills/create-game/SKILL.md` is the step-by-step process for
  an AI assistant. Both are plain markdown, worth reading whatever tools you use.

Every change to a file a package ships (`*.boardlang`, `rules.md`, `README.md`,
`assets/**`, `boardlang.json`) needs a `version` bump in that package's
`boardlang.json`. `npm run check` fails without it.

## Publishing

Merging to `main` publishes. After CI passes on `main`, the **Publish** workflow
submits every package version the registry does not have yet, modules before
the games that use them, and a version already published is skipped. Only
maintainers can merge, and only the workflow holds the publishing token.

Publishing is switched off until the catalog has landed.

If a publish run fails with "GAMEWEAVE_TOKEN is invalid, expired, or from the
wrong account", a maintainer rotates the token: create a new publish-scoped
token from the publishing account, replace the `GAMEWEAVE_TOKEN` secret of the
`production` environment, revoke the old token, and re-run the workflow.

## License

The game definitions, rules and assets in this repository are licensed under
the [Apache License 2.0](LICENSE), except where a package declares otherwise for
its assets. The Gameweave name and marks are not licensed; `NOTICE` and
`TRADEMARKS.md` will say exactly which files that covers when the catalog lands.

See also the [Code of Conduct](CODE_OF_CONDUCT.md) and the
[security policy](SECURITY.md).
