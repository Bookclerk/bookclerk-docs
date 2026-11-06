# bookclerk_plugin_e2e

Support for the staged first-party plugin end-to-end suite.

The suite (`tests/staged_plugins.rs`) spawns every guest from a real
installation: platform guests under `$BOOKCLERK_FILES_DIR/plugins/`
(`cargo install-platform`) and optional/example guests under
`$BOOKCLERK_PLUGIN_ARTIFACTS` (`cargo stage-plugins`). It lives in its own
package so ordinary `bookclerk-plugin-host` test runs never need that
installation.

`tests/native_gateway.rs` is separate: it needs no staged installation and
launches the test-only `native_gateway_probe` bin through the production
native-behind-workerd path (CI check `native_gateway`, all three OSes).

| Variable | Effect |
| --- | --- |
| `BOOKCLERK_PLUGIN_ARTIFACTS` / `BOOKCLERK_FILES_DIR` | Staging roots; unset skips locally |
| `BOOKCLERK_STAGED_PLUGINS=a,b` | Check only these ids (plus platform guests); unset checks all |
| `BOOKCLERK_REQUIRE_STAGED_PLUGINS=1` | Missing roots or ledgers fail instead of skipping (CI) |

| Item | Kind | Summary |
| --- | --- | --- |
| [`StagedScope`](#stagedscope) | enum | Which staged guests a run must check. |
| [`all_expected_plugins`](#all_expected_plugins) | function | Every first-party plugin id the full installation must contain. |
| [`require_staged`](#require_staged) | function | Whether missing staging prerequisites must fail the run. |
| [`skip_or_fail`](#skip_or_fail) | function | Skips (local) or panics (required mode) for a missing prerequisite. |
| [`staged_plugin_dir`](#staged_plugin_dir) | function | Finds the staged directory whose `plugin.toml` declares `id`. |
| [`PLATFORM_PLUGINS`](#platform_plugins) | constant | Platform guests installed into `$BOOKCLERK_FILES_DIR/plugins/`; always checked. |
| [`OPTIONAL_PLUGINS`](#optional_plugins) | constant | Optional first-party guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`. |
| [`EXAMPLE_PLUGINS`](#example_plugins) | constant | Reference Echo guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`. |
| [`STAGED_PLUGINS_ENV`](#staged_plugins_env) | constant | Environment variable selecting a subset of staged guests to check. |
| [`REQUIRE_STAGED_ENV`](#require_staged_env) | constant | Environment variable turning missing staging prerequisites into failures. |

## Enums

### StagedScope

```rust
pub enum StagedScope {
    Full,
    Subset(BTreeSet<String>),
}
```

Which staged guests a run must check.

**Variants**

- `Full`: The complete installation (default).
- `Subset`: Only these non-platform ids; platform guests are always included.

#### StagedScope::parse

```rust
pub fn parse(raw: Option<&str>) -> Result<Self, String>
```

Parses a `BOOKCLERK_STAGED_PLUGINS` value.

##### Arguments

* `raw` - Comma-separated plugin ids; `None`, empty, or `all` means full.

##### Returns

The scope to check.

##### Errors

Returns a message naming the first id that is not a known first-party
plugin, so a typo cannot silently shrink coverage.

#### StagedScope::from_env

```rust
pub fn from_env() -> Result<Self, String>
```

Reads the scope from `BOOKCLERK_STAGED_PLUGINS`.

##### Returns

The parsed scope.

##### Errors

Same as [`StagedScope::parse`](#stagedscopeparse).

#### StagedScope::includes

```rust
pub fn includes(&self, id: &str) -> bool
```

Whether `id` is checked in this scope.

##### Arguments

* `id` - Plugin manifest id.

##### Returns

`true` for platform guests and for ids inside the scope.

#### StagedScope::required_ids

```rust
pub fn required_ids(&self) -> Vec<String>
```

Ids that must be discovered for this scope.

##### Returns

Platform ids followed by the scoped ids (all expected ids when full).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### all_expected_plugins

```rust
pub fn all_expected_plugins() -> Vec<&'static str>
```

Every first-party plugin id the full installation must contain.

#### Returns

Platform, optional, then example ids.

### require_staged

```rust
pub fn require_staged() -> bool
```

Whether missing staging prerequisites must fail the run.

#### Returns

`true` when `BOOKCLERK_REQUIRE_STAGED_PLUGINS=1`.

### skip_or_fail

```rust
pub fn skip_or_fail(reason: &str)
```

Skips (local) or panics (required mode) for a missing prerequisite.

#### Arguments

* `reason` - Human-readable prerequisite that is missing.

#### Panics

Panics when [`require_staged`](#require_staged) is true.

### staged_plugin_dir

```rust
pub fn staged_plugin_dir(root: &Path, id: &str) -> Option<PathBuf>
```

Finds the staged directory whose `plugin.toml` declares `id`.

Staging writes `pk-*` directories keyed by plugin key, so an id cannot be
joined onto the artifacts root directly.

#### Arguments

* `root` - Staging root (`$BOOKCLERK_PLUGIN_ARTIFACTS`).
* `id` - Plugin manifest id.

#### Returns

The guest directory, or `None` when no staged manifest declares `id`.

## Constants

### PLATFORM_PLUGINS

```rust
pub const PLATFORM_PLUGINS: &[&str] = _;
```

Platform guests installed into `$BOOKCLERK_FILES_DIR/plugins/`; always checked.

### OPTIONAL_PLUGINS

```rust
pub const OPTIONAL_PLUGINS: &[&str] = _;
```

Optional first-party guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`.

### EXAMPLE_PLUGINS

```rust
pub const EXAMPLE_PLUGINS: &[&str] = _;
```

Reference Echo guests staged under `$BOOKCLERK_PLUGIN_ARTIFACTS`.

### STAGED_PLUGINS_ENV

```rust
pub const STAGED_PLUGINS_ENV: &str = "BOOKCLERK_STAGED_PLUGINS";
```

Environment variable selecting a subset of staged guests to check.

### REQUIRE_STAGED_ENV

```rust
pub const REQUIRE_STAGED_ENV: &str = "BOOKCLERK_REQUIRE_STAGED_PLUGINS";
```

Environment variable turning missing staging prerequisites into failures.
