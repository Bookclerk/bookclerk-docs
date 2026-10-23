**bookclerk_plugin_catalog > version**

# Module: version

## Contents

**Structs**

- [`Version`](#version) - Parsed `major.minor.patch` with optional pre-release / build metadata.

**Functions**

- [`max_version`](#max_version) - Prefer the greatest release version; if only pre-releases exist, the
- [`newest_newer_than`](#newest_newer_than) - Newest version strictly greater than `current`, if any.

---

## bookclerk_plugin_catalog::version::Version

*Struct*

Parsed `major.minor.patch` with optional pre-release / build metadata.

Ordering follows SemVer 2.0 for the subset we care about: a release is
greater than any pre-release with the same major/minor/patch
(`1.0.0` > `1.0.0-beta`). Pre-release identifiers are compared
lexicographically after numeric segments where both sides are integers.

**Fields:**
- `major: u64` - Semver major component.
- `minor: u64` - Semver minor component.
- `patch: u64` - Semver patch component.
- `pre: Option<String>` - `None` means a release (no `-pre` suffix). Empty vs missing is not used.

**Methods:**

- `fn parse(s: &str) -> Option<Self>` - Parse a version string (`1.2.3`, `1.2.3-beta`, `v1.2.3`).
- `fn is_prerelease(self: &Self) -> bool` - Returns true when this version has a prerelease suffix.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Version`
- **PartialEq**
  - `fn eq(self: &Self, other: &Version) -> bool`
- **PartialOrd**
  - `fn partial_cmp(self: &Self, other: &Self) -> Option<Ordering>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Ord**
  - `fn cmp(self: &Self, other: &Self) -> Ordering`



## bookclerk_plugin_catalog::version::max_version

*Function*

Prefer the greatest release version; if only pre-releases exist, the
greatest pre-release. Falls back to lexical max when nothing parses.

```rust
fn max_version<'a, impl IntoIterator<Item = &'a str>>(versions: impl Trait) -> Option<&'a str>
```



## bookclerk_plugin_catalog::version::newest_newer_than

*Function*

Newest version strictly greater than `current`, if any.

Prefer a newer release over a newer pre-release when both qualify.

```rust
fn newest_newer_than<'a, impl IntoIterator<Item = &'a str>>(current: &str, versions: impl Trait) -> Option<&'a str>
```



