# bookclerk_plugin_abi::package_version

`plugin.toml` `version` as a semantic version.

Every plugin declares a semver `version`. The host pins the `[[settings]]`
schema per plugin key together with the version that produced it, and
orders versions to decide whether a host may re-pin (newer), must refuse
to spawn (older), or is unchanged.

| Item | Kind | Summary |
| --- | --- | --- |
| [`parse_package_version`](#parse_package_version) | function | Parses `raw` as a semantic version. |
| [`package_version_order`](#package_version_order) | function | Orders a host's local package version against the pinned one. |

## Functions

### parse_package_version

```rust
pub fn parse_package_version(raw: &str) -> Result<Version, String>
```

Parses `raw` as a semantic version.

#### Errors

Returns an operator-facing message when `raw` is empty or is not semver.

### package_version_order

```rust
pub fn package_version_order(local: &str, pinned: &str) -> Result<Ordering, String>
```

Orders a host's local package version against the pinned one.

A pinned version that is empty or does not parse counts as `0.0.0`, so a
host with a real version repairs the pin instead of refusing forever.

#### Errors

Returns an error when `local` is not a semantic version.
