# CLAUDE.md

This repository is the open catalog of Gameweave games, written in boardlang
and licensed Apache-2.0. It holds packages and the tooling that checks and
publishes them — no application or engine code. The `gameweave` CLI
(`devDependencies`, pinned exactly) is the whole toolchain.

## Layout

- `games/<name>/` — a game: `<name>.boardlang`, `rules.md`, `boardlang.json`,
  `tests/**/*.boardlang`, `assets/`
- `modules/<name>/` — a module games import (decks, dice, shared queries)
- `gameweave.workspace.json` — declares the workspace: package globs and the
  license policy (`Apache-2.0`, plus CC asset licenses for dual-licensed art)
- `.github/workflows/boardlang-ci.yml` — the reusable CI workflow, also called by
  other gameweave workspaces pinned to a commit SHA; `ci.yml` calls it here;
  `publish.yml` publishes from `main` (switched off by the `PUBLISH_ENABLED`
  repository variable until enabled by a maintainer)

## Commands

```bash
npm ci
npm run validate          # gameweave validate --all
npm test                  # gameweave test --all
npm run check             # gameweave check --all
npm run format            # prettier: markdown, JSON, YAML (never .boardlang)
npx gameweave <validate|test|check> --package <a,b>       # some packages
npx gameweave <validate|test|check> --changed-since origin/main
npx gameweave simulate --game games/<g>/<g>.boardlang --modules modules
npx gameweave fmt games/<g>  &&  npx gameweave lint games/<g>
```

CI runs `validate`, `test` and `check` with `--changed-since` the base branch on
a pull request, `--all` on `main`, then `npm run format:check`.

## Rules

- **Read `.claude/rules/creating-games.md` before adding or changing a game or
  module**, and use the `create-game` skill
  (`.claude/skills/create-game/SKILL.md`) to create one.
- **Bump `version` in `boardlang.json` for every change to a shipped file**
  (`*.boardlang`, `rules.md`, `README.md`, `assets/**`, `boardlang.json`).
  Publishing skips a version the registry already has; `check` fails the
  change (`CLI102`).
- **Every package is `Apache-2.0`** (or dual with a CC asset license). Never add
  an `UNLICENSED` package, and never depend on one.
- **No Gameweave name, logo or diamond mark in any asset.**
- **All game logic is boardlang; every board is a `view` block.** Never set
  `display.type` on a new game.
- **Never publish by hand, and never edit `publish.yml`'s enable conditions to
  get a run.** Merging to `main` is how a package publishes.
- Pin third-party actions to a full commit SHA with a version comment.
