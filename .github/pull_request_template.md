## What changed

<!-- The games or modules this touches, and what is different for a player. -->

## Checklist

- [ ] Every package with a changed shipped file (`*.boardlang`, `rules.md`,
      `README.md`, `assets/**`, `boardlang.json`) has a `version` bump in its
      `boardlang.json` — patch for copy and art, minor or major for gameplay.
      `gameweave check` fails without it.
- [ ] `license` is `Apache-2.0`, or a dual
      `{ "source": "Apache-2.0", "assets": "<CC id>" }` for third-party art,
      with an attribution section in the package README.
- [ ] Every asset is my own work or carries a compatible license, and none
      shows the Gameweave name, logo or diamond mark.
- [ ] `npm run validate && npm test && npm run check` pass locally.
- [ ] For a new game: `npx gameweave simulate --game games/<name>/<name>.boardlang --modules modules`
      reaches game over.
