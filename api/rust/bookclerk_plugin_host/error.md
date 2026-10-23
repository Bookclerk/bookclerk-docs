**bookclerk_plugin_host > error**

# Module: error

## Contents

**Enums**

- [`PluginError`](#pluginerror) - Failures from discovery, RPC, consent, or plugin adapters.

**Type Aliases**

- [`Result`](#result) - Result alias for [`PluginError`](error.md#bookclerk_plugin_hosterrorpluginerror).

---

## bookclerk_plugin_host::error::PluginError

*Enum*

Failures from discovery, RPC, consent, or plugin adapters.

**Variants:**
- `Message(String)` - Operator-facing error text with no structured code.
- `InstanceSelection(String)` - Instance selection failed closed: missing, wrong plugin key, or ambiguous.
- `Conflict(String)` - The request conflicts with cluster state (an alias owned by another
- `NotFound(String)` - A plugin, version, or row the request names does not exist.
- `Unavailable(String)` - Backend or plugin transport is temporarily unreachable; retry the same
- `Io(std::io::Error)` - Filesystem or process I/O failure.
- `Json(serde_json::Error)` - JSON encode/decode failure on the RPC wire or grant file.
- `Toml(toml::de::Error)` - TOML decode failure (settings tables, legacy paths).
- `Manifest(bookclerk_plugin_manifest::Error)` - Invalid or unsupported `plugin.toml` / package manifest.
- `Other(anyhow::Error)` - Catch-all for otherwise unclassified failures.
- `Abi{ code: String, message: String }` - Guest ABI error with a preserved wire `code` (including `unknown`).

**Methods:**

- `fn message<impl Into<String>>(msg: impl Trait) -> Self` - Builds a `PluginError::Message` from any displayable string.
- `fn conflict<impl Into<String>>(msg: impl Trait) -> Self` - Builds a `PluginError::Conflict`.
- `fn unavailable<impl Into<String>>(msg: impl Trait) -> Self` - Builds a structured unavailable error (lost RPC, timeout, incomplete reply).
- `fn instance_selection<impl Into<String>>(msg: impl Trait) -> Self` - Instance id, plugin key, or alias cannot select exactly one instance.
- `fn is_instance_selection(self: &Self) -> bool` - True when spawn must stop instead of skipping this guest.
- `fn from_abi<impl Into<String>>(code: Option<&str>, message: impl Trait) -> Self` - Maps a guest ABI error, preserving `unavailable`
- `fn is_ambiguous_transport(self: &Self) -> bool` - True when an atomic-execute caller should retry the same operation id.

**Trait Implementations:**

- **From**
  - `fn from(source: toml::de::Error) -> Self`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`
- **From**
  - `fn from(source: serde_json::Error) -> Self`
- **From**
  - `fn from(source: anyhow::Error) -> Self`
- **From**
  - `fn from(source: std::io::Error) -> Self`
- **From**
  - `fn from(source: bookclerk_plugin_manifest::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_host::error::Result

*Type Alias*: `std::result::Result<T, PluginError>`

Result alias for [`PluginError`](error.md#bookclerk_plugin_hosterrorpluginerror).



