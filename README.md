# semver-nv

A semantic version is a release number of the form `MAJOR.MINOR.PATCH`,
where each of the three numbers says what a caller may assume about the
release. The form, and the order two versions are compared in, are
specified in [Semantic Versioning 2.0.0](https://semver.org). A
requirement is what a manifest writes where a version would go, such as
`^1.2` or `>=1.0.0, <2.0.0`, and it says which releases a program
accepts. This package reads both, orders versions, and answers which
release a requirement chooses. The requirement grammar is the [Orbit
package registry's](https://novo-lang.org/docs/registry/semver.html)
rather than the specification's.

## What a version and a requirement are

A version is three non-negative integers separated by dots. Semantic
Versioning 2.0.0 item 2 gives that form and adds that none of the three
may carry a leading zero. There is no `v` in front of them. A Git tag
may still be called `v1.4.2`, because a tag and a version are different
things.

Two labels may follow the three numbers. A **prerelease label** comes
after a hyphen, as in `1.4.2-rc.1`. It marks a release that is not
ready, and item 9 ranks it below the plain version carrying the same
three numbers. **Build metadata** comes after a plus, as in
`1.4.2+build.7`. It records where a build came from, and item 10 leaves
it out of every comparison, so two versions differing only in build
metadata are one version.

**Precedence** is the order versions are compared in, and item 11
defines it. The three numbers are compared as numbers, left to right,
which is not the order the two strings sort in. `0.10.0` is above
`0.9.0` as a version and below it alphabetically. Precedence is what a
resolver means by the highest release.

Under major version zero the promises are weaker. Item 4 leaves
everything under `0.y.z` free to change at any time. The Orbit registry
narrows that to one rule a caller can rely on. Below `1.0.0` a breaking
change goes in the minor number, so `0.4.9` is compatible with `0.4.1`
and `0.5.0` is not.

A **requirement** is a comma-separated conjunction of the primitives
below. A version answers a requirement only by answering every part of
it.

| Written | Means |
| --- | --- |
| `*` | any release |
| `1.x`, `1.2.x`, `1.2.*` | any version with those leading numbers |
| `^1.2.3` | at least `1.2.3`, below the next version allowed to break a caller |
| `~1.2.3` | at least `1.2.3`, below `1.3.0` |
| `~1` | at least `1.0.0`, below `2.0.0`, because no minor was written and none is held |
| `>=`, `>`, `<=`, `<`, `=` | the comparison, against one version |
| `1.2.3` | a caret requirement, which is what a manifest writing `"1.2"` means |

| Quantity | Value |
| --- | --- |
| Numbers in a version | 3 |
| Optional labels after them | 2 |
| Characters that introduce them | `-` and `+` |
| Passes over the text per parse | 1 |
| Passes over the candidates per `best` | 1 |

## Install

```
novo pkg add semver-nv
```

## Example

```novo
use semver

fn main() [io]
    // A caret refuses the release that is allowed to break a caller.
    // Below 1.0.0 that is the next minor.
    println("${semver.satisfies("^0.4.1", "0.5.0")}")   // false

    // The highest listed release that answers the requirement.
    println(semver.best("^0.4", ["0.4.1", "0.4.9", "0.5.0"]) ?? "-")   // 0.4.9
```

Build and test with `novo pkg build` and `novo test tests/semver_tests.nv`.

## What the package contains

| Module | Contents |
| --- | --- |
| `semver` | The `Version` value and its two questions, `parse` and `compare`, the two calls that answer whether a version satisfies a requirement, `requirement_ok`, and `best`. |

The API reference is on
[the package's page](https://novo-lang.org/packages/semver-nv). `novo
doc` generates it from these sources: every `pub` declaration with its
signature, its effect row and the comment block written above it. Each
entry carries a worked example, and `novo doc` compiles them.

`Version` carries `major`, `minor`, `patch`, `pre` and `build`. `pre`
and `build` are empty strings when the version carried neither, so
there is no optional to unwrap for the common case.

## How to choose an entry point

**`satisfies` takes the requirement and the version as text.** That is
what a manifest and a registry listing each hold, so it is the call for
a program reading either of them.

**`matches` takes a parsed `Version`.** Use it where one version is
asked about several requirements, because the version is then parsed
once rather than once per question.

**`best` takes the requirement and the versions a registry lists.** It
answers the one to install, as the candidate was written. This is the
call a resolver makes.

**`requirement_ok` answers whether a string is a requirement at all.**
`matches`, `satisfies` and `best` all answer no to a requirement
nothing satisfies and no to a string that is not a requirement. Ask
this once where the manifest is read, so a typo is reported as a typo.

**`parse` and `compare` are the version itself.** `parse` reads one and
`compare` orders two. A program that sorts releases or picks the newest
by hand uses these.

## The rules a user needs

1. **Version order is not text order.** The three numbers are compared
   as numbers, so `0.10.0` is above `0.9.0`. Semantic Versioning 2.0.0
   item 11 is the rule. Sorting version strings as text installs the
   wrong release.
2. **A release outranks its own prerelease.** `1.0.0-alpha` is below
   `1.0.0`, per item 11.
3. **Build metadata is ignored in every comparison.** `1.4.2` and
   `1.4.2+ci.881` compare equal, per item 10. An answer of 0 from
   `compare` therefore does not mean the two strings are the same, and
   publishing the second after the first is refused as a duplicate.
4. **Prerelease labels are compared as plain text.** Item 11 compares
   them as dot-separated identifiers, with a numeric identifier
   compared as a number. Plain text orders `alpha` before `beta` before
   `rc`, and it puts `rc.10` before `rc.2`. The Orbit registry compares
   them as plain text as well.
5. **A leading zero is accepted here and refused at publish.** `parse`
   answers `1.2.3` for `01.2.3`. Item 2 forbids the leading zero, and
   the registry lists `01.2.3` among the strings its
   `MALFORMED_VERSION` rule rejects. A version this package accepts can
   still be refused when it is published.
6. **A caret moves its promise down one place below `1.0.0`.** `^0.4.1`
   allows `0.4.9` and refuses `0.5.0`, and `^0.0.3` allows only
   `0.0.3`. That is what makes a `0.x` number carry information.
7. **A prerelease is never picked up by accident.** `^1.0.0` does not
   match `1.1.0-alpha`, and neither does `*`. A prerelease is chosen
   only where the requirement named one at the same three numbers, as
   `^1.1.0-alpha` names `1.1.0-beta`.
8. **A requirement is a conjunction.** Every part of `>=1.2.0, <2.0.0`
   has to hold. Whitespace around the comma is a manifest's layout and
   not a meaning.
9. **A wildcard under an operator is not a requirement.** `>=1.x` names
   no single version to compare against, so `requirement_ok` answers
   false for it.
10. **A `false` does not say which side was malformed.** `satisfies`
    answers false for a version that does not parse and for a
    requirement that does not parse. `parse` and `requirement_ok` are
    what tell those apart.
11. **`best` skips a candidate that does not parse.** Such a candidate
    is not a release the registry could serve, so one stray entry in a
    listing does not stop a resolve.
12. **`matches` parses the requirement on every call.** A caller asking
    many versions about one requirement pays that parse each time.

## What is not included

- **The refusal of a leading zero.** `01.2.3` parses here and is
  refused by Semantic Versioning 2.0.0 item 2 and by the registry. A
  caller about to publish checks that separately, and rule 5 above
  states the consequence.
- **Item 11's dot-separated identifier comparison.** Prerelease labels
  are compared as plain text, which orders the labels in common use and
  misorders a numeric identifier past nine.
- **Version arithmetic.** Nothing here bumps a version or suggests the
  next one.
- **Hyphen ranges and alternation.** `1.2.3 - 2.3.4` and `||` are in
  other requirement grammars and not in the Orbit registry's. A
  requirement this package accepted and the registry did not would be a
  build that works locally and fails on install.
- **Resolution across a dependency graph.** `best` answers for one
  package. Choosing versions that satisfy every package at once is the
  resolver's work.
- **A parsed requirement a caller can keep.** `matches` parses its
  requirement on every call. Keeping one alive would put a type in the
  API that only a caller in a loop needs.
- **Any input or output.** Every function here is a decision over text
  the caller already holds, which is why the layer is `core` and no
  function declares an effect.

## Related packages

- [changelog-nv](https://novo-lang.org/packages/changelog-nv) reads a
  `CHANGELOG.md` into releases and derives the next version number from
  a range of commits. It takes its version type from this package, so
  two releases compare the same way in a changelog and at publish time.
- [toml-nv](https://novo-lang.org/packages/toml-nv) reads the manifest
  a requirement is written in. It answers the text of a dependency's
  requirement, and this package says what that text means.

## Tests

```
novo test tests/semver_tests.nv
```

The vectors are the rules the Orbit registry publishes. The suite
asserts the forms that parse and the forms its `MALFORMED_VERSION` rule
refuses, the order of the three numbers, that a release outranks its
prerelease, that prerelease labels compare as plain text, that build
metadata is left out of a comparison, the caret rule above and below
`1.0.0`, the tilde rule with and without a minor written, the five
comparison operators, a conjunction, the wildcards, a bare version as a
caret requirement, the prerelease rule, and a requirement that is not
one. Four cases cover `best`: the highest release that answers, a
listing ordered as versions rather than as text, a listing with a stray
entry in it, and an empty listing.

Every example in a documentation comment is compiled by `novo doc` and
run by `novo test`, so an example that has stopped being true is a
failing test.

## Licence

Apache-2.0. See `LICENSE`.

<!-- docs/writing-a-readme.md is the style guide for this page. -->
