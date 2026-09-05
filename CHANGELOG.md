# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

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
