**bookclerk_plugin_abi > package_version**

# Module: package_version

## Contents

**Functions**

- [`package_version_order`](#package_version_order) - Orders a host's local package version against the pinned one.
- [`parse_package_version`](#parse_package_version) - Parses `raw` as a semantic version.

---

## bookclerk_plugin_abi::package_version::package_version_order

*Function*

Orders a host's local package version against the pinned one.

A pinned version that is empty or does not parse counts as `0.0.0`, so a
host with a real version repairs the pin instead of refusing forever.

# Errors

Returns an error when `local` is not a semantic version.

```rust
fn package_version_order(local: &str, pinned: &str) -> Result<std::cmp::Ordering, String>
```



## bookclerk_plugin_abi::package_version::parse_package_version

*Function*

Parses `raw` as a semantic version.

# Errors

Returns an operator-facing message when `raw` is empty or is not semver.

```rust
fn parse_package_version(raw: &str) -> Result<semver::Version, String>
```



