**bookclerk_plugin_e2e**

# Module: bookclerk_plugin_e2e

## Contents

**Enums**

- [`StagedScope`](#stagedscope) - Which staged guests a run must check.

**Functions**

- [`all_expected_plugins`](#all_expected_plugins) - Every first-party plugin id the full installation must contain.
- [`require_staged`](#require_staged) - Whether missing staging prerequisites must fail the run.
- [`skip_or_fail`](#skip_or_fail) - Skips (local) or panics (required mode) for a missing prerequisite.
- [`staged_plugin_dir`](#staged_plugin_dir) - Finds the staged directory whose `plugin.toml` declares `id`.

**Constants**

- [`EXAMPLE_PLUGINS`](#example_plugins) - Reference Echo guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`.
- [`OPTIONAL_PLUGINS`](#optional_plugins) - Optional first-party guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`.
- [`PLATFORM_PLUGINS`](#platform_plugins) - Platform guests installed into `$BOOKCLERK_FILES_DIR/plugins/`; always checked.
- [`REQUIRE_STAGED_ENV`](#require_staged_env) - Environment variable turning missing staging prerequisites into failures.
- [`STAGED_PLUGINS_ENV`](#staged_plugins_env) - Environment variable selecting a subset of staged guests to check.

---

## bookclerk_plugin_e2e::EXAMPLE_PLUGINS

*Constant*: `&[&str]`

Reference Echo guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`.



## bookclerk_plugin_e2e::OPTIONAL_PLUGINS

*Constant*: `&[&str]`

Optional first-party guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`.



## bookclerk_plugin_e2e::PLATFORM_PLUGINS

*Constant*: `&[&str]`

Platform guests installed into `$BOOKCLERK_FILES_DIR/plugins/`; always checked.



## bookclerk_plugin_e2e::REQUIRE_STAGED_ENV

*Constant*: `&str`

Environment variable turning missing staging prerequisites into failures.



## bookclerk_plugin_e2e::STAGED_PLUGINS_ENV

*Constant*: `&str`

Environment variable selecting a subset of staged guests to check.



## bookclerk_plugin_e2e::StagedScope

*Enum*

Which staged guests a run must check.

**Variants:**
- `Full` - The complete installation (default).
- `Subset(std::collections::BTreeSet<String>)` - Only these non-platform ids; platform guests are always included.

**Methods:**

- `fn parse(raw: Option<&str>) -> Result<Self, String>` - Parses a `BOOKCLERK_STAGED_PLUGINS` value.
- `fn from_env() -> Result<Self, String>` - Reads the scope from `BOOKCLERK_STAGED_PLUGINS`.
- `fn includes(self: &Self, id: &str) -> bool` - Whether `id` is checked in this scope.
- `fn required_ids(self: &Self) -> Vec<String>` - Ids that must be discovered for this scope.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &StagedScope) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> StagedScope`



## bookclerk_plugin_e2e::all_expected_plugins

*Function*

Every first-party plugin id the full installation must contain.

# Returns

Platform, optional, then example ids.

```rust
fn all_expected_plugins() -> Vec<&'static str>
```



## bookclerk_plugin_e2e::require_staged

*Function*

Whether missing staging prerequisites must fail the run.

# Returns

`true` when `BOOKCLERK_REQUIRE_STAGED_PLUGINS=1`.

```rust
fn require_staged() -> bool
```



## bookclerk_plugin_e2e::skip_or_fail

*Function*

Skips (local) or panics (required mode) for a missing prerequisite.

# Arguments

* `reason` - Human-readable prerequisite that is missing.

# Panics

Panics when [`require_staged`](bookclerk_plugin_e2e.md#bookclerk_plugin_e2erequire_staged) is true.

```rust
fn skip_or_fail(reason: &str)
```



## bookclerk_plugin_e2e::staged_plugin_dir

*Function*

Finds the staged directory whose `plugin.toml` declares `id`.

Staging writes `pk-*` directories keyed by plugin key, so an id cannot be
joined onto the artifacts root directly.

# Arguments

* `root` - Staging root (`$BOOKCLERK_PLUGIN_ARTIFACTS`).
* `id` - Plugin manifest id.

# Returns

The guest directory, or `None` when no staged manifest declares `id`.

```rust
fn staged_plugin_dir(root: &std::path::Path, id: &str) -> Option<std::path::PathBuf>
```



