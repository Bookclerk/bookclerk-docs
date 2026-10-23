**bookclerk_migrate > settings**

# Module: settings

## Contents

**Structs**

- [`ClassicPluginSettings`](#classicpluginsettings) - Classic settings that belong to plugin instances rather than `config.toml`.

**Functions**

- [`apply_settings_json`](#apply_settings_json) - Apply classic Settings.json keys onto `config`.
- [`config_to_settings_json`](#config_to_settings_json) - Best-effort reverse of [`apply_settings_json`](settings.md#bookclerk_migratesettingsapply_settings_json) for Libation Settings.json export.
- [`load_settings_json`](#load_settings_json) - Load Settings.json as a JSON object.

---

## bookclerk_migrate::settings::ClassicPluginSettings

*Struct*

Classic settings that belong to plugin instances rather than `config.toml`.

**Fields:**
- `books_root: Option<std::path::PathBuf>` - Libation `Books` directory: the `local` destination's `root`.
- `audible_bitrate: Option<String>` - Libation `FileDownloadQuality` as the Audible plugin's `bitrate`

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ClassicPluginSettings) -> bool`
- **Default**
  - `fn default() -> ClassicPluginSettings`
- **Clone**
  - `fn clone(self: &Self) -> ClassicPluginSettings`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_migrate::settings::apply_settings_json

*Function*

Apply classic Settings.json keys onto `config`.

Keys that configure a plugin (the books directory, the Audible bitrate)
are returned instead so the caller can write them on the plugin instance.

# Arguments

* `config` - Loaded Bookclerk configuration.
* `settings` - `settings` input for this call.

```rust
fn apply_settings_json(config: & mut bookclerk_config::Config, settings: &serde_json::Value) -> ClassicPluginSettings
```



## bookclerk_migrate::settings::config_to_settings_json

*Function*

Best-effort reverse of [`apply_settings_json`](settings.md#bookclerk_migratesettingsapply_settings_json) for Libation Settings.json export.

# Arguments

* `config` - Loaded Bookclerk configuration.
* `plugin_settings` - Books directory and Audible bitrate from the plugin
  instances, when the caller could read them.

# Returns

`Value` result.

```rust
fn config_to_settings_json(config: &bookclerk_config::Config, plugin_settings: &ClassicPluginSettings) -> serde_json::Value
```



## bookclerk_migrate::settings::load_settings_json

*Function*

Load Settings.json as a JSON object.

# Arguments

* `path` - Filesystem path involved in this operation.

# Returns

On success, the inner `Value` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn load_settings_json(path: &std::path::Path) -> crate::error::Result<serde_json::Value>
```



