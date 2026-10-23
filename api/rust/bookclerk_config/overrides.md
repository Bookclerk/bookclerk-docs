**bookclerk_config > overrides**

# Module: overrides

## Contents

**Functions**

- [`apply_config_updates`](#apply_config_updates) - Load config, apply `pairs`, and write the result back to the resolved config path.
- [`apply_config_updates_from_path`](#apply_config_updates_from_path) - Load `path`, apply `pairs`, and write the updated TOML back to the same path.
- [`apply_setting_overrides`](#apply_setting_overrides) - Apply classic-style `-o Setting=value` overrides onto `config`.

---

## bookclerk_config::overrides::apply_config_updates

*Function*

Load config, apply `pairs`, and write the result back to the resolved config path.

# Arguments

* `files_dir` - Optional `BOOKCLERK_FILES_DIR` override.
* `config_path` - Optional explicit `config.toml` path.
* `pairs` - `(key, value)` overrides (classic or dotted).

# Returns

The updated [`Config`](settings.md#bookclerk_configsettingsconfig) after a successful write.

# Errors

Propagates load, validation, and write failures as [`crate::ConfigError`](error.md#bookclerk_configerrorconfigerror).

```rust
fn apply_config_updates(files_dir: Option<std::path::PathBuf>, config_path: Option<std::path::PathBuf>, pairs: &[(&str, &str)]) -> crate::Result<crate::Config>
```



## bookclerk_config::overrides::apply_config_updates_from_path

*Function*

Load `path`, apply `pairs`, and write the updated TOML back to the same path.

# Arguments

* `path` - Existing `config.toml` to mutate in place.
* `pairs` - `(key, value)` overrides (classic or dotted).

# Errors

Propagates read, parse, and write failures as [`crate::ConfigError`](error.md#bookclerk_configerrorconfigerror).

```rust
fn apply_config_updates_from_path(path: &std::path::Path, pairs: &[(&str, &str)]) -> crate::Result<crate::Config>
```



## bookclerk_config::overrides::apply_setting_overrides

*Function*

Apply classic-style `-o Setting=value` overrides onto `config`.

Keys may be classic PascalCase (`UseWidevine`) or dotted TOML paths
(`output.widevine`). Plugin settings are not `config.toml` keys; they are
written on the plugin instance (`bookclerk plugins config set`).

```rust
fn apply_setting_overrides(config: & mut crate::Config, pairs: &[(&str, &str)])
```



