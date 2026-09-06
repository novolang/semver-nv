# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

## 0.1.2

Developed in its own repository from this version.  `novolang/semver-nv` is
where the sources live, where CI runs and where releases are tagged;
the novo-lang monorepo no longer carries a copy.  No code changed —
every signature and every byte on the wire is what 0.1.1 shipped.

- **`LICENSE` ships with the package.**  It is on the publish
  allow-list, so the tarball now carries the Apache-2.0 text rather
  than only naming it in the manifest.

## 0.1.1

A patch: the same grammar and the same answers.  The differential
against the Orbit registry's own requirement parser still agrees on
every case.

- **The test module moved out of `src/`.**  A package's `src/` ships
  whole and a consumer compiles every module in it, so the suite is
  under `tests/` where it is not published.  Run it with
  `novo test tests/semver_tests.nv`.
- **The manifest carries the fields the registry browses by.**
  `category`, `tags`, `repository` and `maintainers` were added after
  `0.1.0` was published, and a published version is never replaced, so
  this release is the first one the packages page can shelve and
  filter.

There is no bit arithmetic here to rewrite onto the new operators: a
version is three numbers and two strings, and comparing them is
ordinary comparison.

## 0.1.0

First release: `parse`, `compare`, `matches`, `satisfies`,
`requirement_ok`, `best`, and the `Version` value's `text` and
`is_prerelease`.

- **The registry's rules, mirrored.**  Parsing and ordering follow the
  Orbit registry's own parser, and a differential run compares the two
  over a corpus of 24 versions and all 576 pairs, asserted identical.
- **The whole requirement grammar** a manifest may write: `^`, `~`,
  `>=`, `>`, `<=`, `<`, `=`, wildcards, a bare version as a caret
  requirement, and a comma-separated conjunction of any of them.
- **Caret moves its promise down one place below `1.0.0`**, so `^0.4.1`
  refuses `0.5.0` — the rule that makes `0.x` mean something.
- **A prerelease is never picked up by accident.**  It is chosen only
  when the requirement named one at the same three numbers.
- **`best`** answers the resolver's question directly: the highest
  candidate that satisfies a requirement, ordered as versions rather
  than as text.
