**bookclerk_plugin_tools**

# Module: bookclerk_plugin_tools

## Contents

**Functions**

- [`fmt_plugin_toml`](#fmt_plugin_toml) - Formats `plugin.toml` text to the canonical serialized form.
- [`load_manifest`](#load_manifest) - Parses and validates `plugin.toml` text without rewriting it.

---

## bookclerk_plugin_tools::fmt_plugin_toml

*Function*

Formats `plugin.toml` text to the canonical serialized form.

Parses with the manifest crate, then re-emits a stable key order / spacing
suitable for `bookclerk-plugin fmt` and CI `--check` diffs.

# Arguments

* `text` - Raw contents of a `plugin.toml` file.

# Returns

Canonical TOML string (typically ending with a trailing newline).

# Errors

Returns a manifest parse/format error when the text is invalid TOML or
fails schema validation.

```rust
fn fmt_plugin_toml(text: &str) -> Result<String, bookclerk_plugin_manifest::Error>
```



## bookclerk_plugin_tools::load_manifest

*Function*

Parses and validates `plugin.toml` text without rewriting it.

# Arguments

* `text` - Raw contents of a `plugin.toml` file.

# Returns

Structured [`PluginManifest`](../bookclerk_plugin_manifest/types.md#bookclerk_plugin_manifesttypespluginmanifest) ready for inspection.

# Errors

Returns a manifest error when parsing or validation fails.

```rust
fn load_manifest(text: &str) -> Result<bookclerk_plugin_manifest::PluginManifest, bookclerk_plugin_manifest::Error>
```



