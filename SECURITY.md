# Security Policy

## Reporting a vulnerability

Email **security@gameweave.com**. Please do not open a public issue, pull
request or discussion for a security problem.

Include what you found, where (a path in this repository, a package name and
version on the registry, or a page on gameweave.com), and the steps that show
it. We will acknowledge your report, keep you informed while we work on it, and
credit you when the fix ships unless you would rather we did not.

## What counts

This repository holds boardlang game definitions, their rules and their art,
and the tooling that checks and publishes them. Reports we want include:

- a game or module that makes the engine, the registry or the web app misbehave
  in a way its author should not be able to cause — reading another player's
  hidden information, crashing a table, or running code a player did not choose
  to run;
- an asset carrying active content (script in an SVG, for example) that reaches
  a player;
- a credential or token committed to this repository;
- a weakness in the CI or publish workflows under `.github/workflows/` that
  would let a pull request reach the publishing token or publish a package.

The `gameweave` command-line tool, the engine and the registry are developed
elsewhere, but report problems in them to the same address.

A game that is merely unbalanced, or a rules bug, is not a security issue —
open an ordinary issue for it.
