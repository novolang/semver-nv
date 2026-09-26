# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**.  See [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

## 0.1.6 — 2026-09-26

- **The toolchain floor is 0.12.0**, where the manifest named none.
  `best` and the requirement parser hand an absent value back with `!`
  on an optional, a 0.12.0 form. An older toolchain reads `!` on an
  optional as unwrap or panic, so it would panic where this release
  answers `None`. No signature and no answer changed.
- **The test suite compiles on 0.12.0.** The sorting test copied its
  list with `var pool = shuffled`, and 0.12.0 refuses a writable name
  made from a `let` list (E2038). It now takes a copy, `shuffled[:]`,
  and the 21 tests pass.

## 0.1.5 — 2026-09-18

The documentation and comments in plain prose; no signature changed.

- **The leading zero is documented where a reader meets it.**  `parse`
  accepts `01.2.3` and answers the version `1.2.3`.  Semantic
  Versioning 2.0.0 item 2 forbids a leading zero, and the Orbit
  registry lists `01.2.3` among the strings its `MALFORMED_VERSION`
  rule refuses, so a version this package accepts can still be refused
  at publish.  The README, the module header, the documentation above
  `parse` and the comment on the digit check each say so.  No code
  changed.

## 0.1.4 — 2026-09-08

- **The layer is declared.**  `layer = "core"` in the manifest.  The
  public API requires no effects, and `novo pkg publish` checks the
  code against that layer.  No code changed.  The layers are described
  under Design in the [publishing
  guide](https://novo-lang.org/docs/publishing.html#design).
- **The sources are in the canonical form** `novo fmt` prints today.
  Spacing and alignment only, and no code changed.

## 0.1.3

The reference generated from the code, with the examples in it run as
tests.  No code changed, and every requirement answers what it answered
in 0.1.2.

- **Every `pub` item is documented under Go's rule**, the comment block
  directly above the declaration, its first sentence the summary a
  reader meets before opening anything.  `Version.text` and
  `Version.is_prerelease` carry their own.  `novo doc` turns the lot
  into [the package's page](https://novo-lang.org/packages/semver-nv).
- **Eight worked examples, and they run.**  The rules a caller has to
  know before reading a `false` are shown rather than described.  A
  caret moves its promise down one place below `1.0.0`, a prerelease is
  never picked up by accident, build metadata makes two versions
  compare equal, and `0.4.10` outranks `0.4.2` though the text says
  otherwise.  A fenced `novo` block in a documentation comment is
  compiled by `novo doc` and run by `novo test src/semver.nv`.

## 0.1.2

The Apache-2.0 text in the tarball.  No code changed, and every
signature is what 0.1.1 shipped.

- **`LICENSE` ships with the package.**  It is on the publish
  allow-list, so the tarball carries the licence text rather than only
  naming it in the manifest.

## 0.1.1

A patch, with the same grammar and the same answers.

- **The test module moved out of `src/`.**  A package's `src/` ships
  whole and a consumer compiles every module in it, so the suite is
  under `tests/` where it is not published.  Run it with
  `novo test tests/semver_tests.nv`.
- **The manifest carries the fields the registry browses by.**
  `category`, `tags`, `repository` and `maintainers` were added after
  `0.1.0` was published, and a published version is never replaced, so
  this release is the first one the packages page can shelve and
  filter.

## 0.1.0

The first release.  `parse`, `compare`, `matches`, `satisfies`,
`requirement_ok`, `best`, and the `Version` value's `text` and
`is_prerelease`.

- **Versions as the Orbit registry orders them.**  Parsing and ordering
  follow the registry's own rules: the three numbers compared as
  numbers, a release above the prerelease carrying the same three, and
  build metadata left out of every comparison.
- **The whole requirement grammar** a manifest may write: `^`, `~`,
  `>=`, `>`, `<=`, `<`, `=`, wildcards, a bare version as a caret
  requirement, and a comma-separated conjunction of any of them.
- **Caret moves its promise down one place below `1.0.0`**, so `^0.4.1`
  refuses `0.5.0`.  That is the rule that makes a `0.x` number carry
  information.
- **A prerelease is never picked up by accident.**  It is chosen only
  where the requirement named one at the same three numbers.
- **`best`** answers the resolver's question directly.  It is the
  highest candidate that satisfies a requirement, ordered as versions
  rather than as text.
