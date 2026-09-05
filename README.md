> Developed in the novo-lang monorepo under `orbit/semver-nv`, which is the source of truth until this package graduates out of it.  This repository is a mirror: it is where CI runs and where releases are tagged, and changes are made upstream.

# semver-nv

Semantic versions: parsing, ordering, and deciding whether a version
answers a requirement. A version is `MAJOR.MINOR.PATCH` with an
optional `-prerelease` and `+build`; a requirement is what a manifest
writes — `^1.2`, `~0.4.1`, `>=1.0.0, <2.0.0`, `=1.4.2`, `1.x`, `*`. The
two together are how a package manager turns "what this program asked
for" into "which release to install".

```novo
use semver

fn main() [io]
    println("${semver.satisfies("^0.4.1", "0.5.0")}")           // false
    println(semver.best("^0.4", ["0.4.1", "0.4.9", "0.5.0"]) ?? "-")  // 0.4.9
```

```
novo pkg add semver-nv
```

## What it gives you

| Function | |
|---|---|
| `semver.parse(text: Str) -> ?Version` | a version, or `None` when the text is not one |
| `semver.compare(a: Version, b: Version) -> Int` | `-1`, `0` or `1`, by the ordering a resolver picks "highest" with |
| `semver.matches(requirement: Str, v: Version) -> Bool` | does this version answer that requirement |
| `semver.satisfies(requirement: Str, version_text: Str) -> Bool` | the same, both sides as text |
| `semver.requirement_ok(requirement: Str) -> Bool` | is that a requirement at all |
| `semver.best(requirement: Str, candidates: [Str]) -> ?Str` | the highest candidate that answers it |
| `v.text() -> Str` | the version as text, parsing back to an equal value |
| `v.is_prerelease() -> Bool` | whether it ranks below the plain version with the same numbers |

`Version` carries `major`, `minor`, `patch`, `pre` and `build`. `pre`
and `build` are empty strings when absent, so there is no optional to
unwrap for the common case.

## The requirement grammar

A requirement is a **comma-separated conjunction**: a version answers it
only by answering every part.

| Written | Means |
|---|---|
| `*` | any release |
| `1.x`, `1.2.x`, `1.2.*` | any version with those leading numbers |
| `^1.2.3` | at least `1.2.3`, below the next version allowed to break you |
| `~1.2.3` | at least `1.2.3`, below `1.3.0` |
| `~1` | at least `1.0.0`, below `2.0.0` — no minor was written, so none is held |
| `>=`, `>`, `<=`, `<`, `=` | the comparison, against one version |
| `1.2.3` | a caret requirement — the convention every manifest that writes `"1.2"` follows |

**Caret moves its promise down one place below `1.0.0`.** In `0.x` a
minor bump is the break, so `^0.4.1` allows `0.4.9` and refuses
`0.5.0`; in `0.0.x` every patch is a break, so `^0.0.3` allows only
`0.0.3`. That is the rule that makes `0.x` mean "still finding its
shape" instead of meaning nothing.

**A prerelease is never picked up by accident.** `^1.0.0` does not
match `1.1.0-alpha`, and neither does `*`. A prerelease is only chosen
when the requirement named one at the same three numbers —
`^1.1.0-alpha` matches `1.1.0-beta`. Without that rule a release nobody
had finished would arrive in a build nobody had changed.

## Ordering

`compare` is the ordering a resolver picks "highest" with, and it is
not text order: `0.10.0` is above `0.9.0` as a version and below it
alphabetically. A release outranks its own prerelease
(`1.0.0-alpha` < `1.0.0`), and build metadata is ignored entirely, so
`1.4.2` and `1.4.2+ci.881` compare equal — which is why publishing the
second after the first is refused as a duplicate.

Two narrowings, both stated rather than hidden:

- **Prerelease labels compare as plain text**, not by semver §11's
  dot-separated identifier rules. `alpha` < `beta` < `rc` works because
  that is also alphabetical order; `rc.10` sorts *before* `rc.2`, so
  write `rc.02` if you expect to pass nine.
- **A leading zero is accepted**: `01.2.3` parses as `1.2.3`.

Both are inherited on purpose. This package mirrors the parser the
Orbit registry runs, and a mirror that quietly disagreed with the thing
it mirrors would be worse than no mirror — a version the registry
accepts must not be one this package rejects.

## What it costs

One pass over the text per parse, one pass over the candidate list per
`best`. No table, no state, and the only allocation is the pieces the
splitting produces. Nothing here is effectful: it is text in, decisions
out.

`matches` parses the requirement on every call. A caller asking many
versions about one requirement pays for that parse each time; the loop
is short and the strings are tiny, and keeping a parsed requirement
alive would put a type in the API that only that caller needs.

## What it does not do

No version *arithmetic* — nothing here bumps a version or suggests the
next one. No hyphen ranges (`1.2.3 - 2.3.4`) and no `||` alternation:
the Orbit registry's grammar has neither, and a requirement this
package accepted but the registry did not would be a build that works
locally and fails on install. No resolution across a dependency graph;
`best` answers for one package and the graph is the resolver's problem.

## Tests

```
novo test tests/semver_tests.nv
```

The vectors are the registry's published rules: every string its
`MALFORMED_VERSION` rule lists, the ordering its `MONOTONIC` rule sorts
by, and each requirement form a manifest may contain. Beside them runs
a differential comparison against the registry backend's own parser
over a shared corpus — 24 versions parsed and all 576 pairs compared,
asserted identical character for character. That is what makes
"mirrors the registry" a measurement rather than a claim.
