**bookclerk_plugin_manifest > logo**

# Module: logo

## Contents

**Enums**

- [`LogoKind`](#logokind) - Classified `plugin.toml` `logo` value after successful validation.

**Functions**

- [`embedded_logo_api_path`](#embedded_logo_api_path) - Builds the Settings / UI href for an embedded logo.
- [`logo_content_type`](#logo_content_type) - Returns the MIME type for an embedded logo path based on its extension.
- [`validate_logo`](#validate_logo) - Validates an optional logo string from `plugin.toml`.

**Constants**

- [`LOGO_EXTENSIONS`](#logo_extensions) - Allowed image extensions for embedded logos (lowercase, including the dot).
- [`MAX_EMBEDDED_LOGO_BYTES`](#max_embedded_logo_bytes) - Maximum byte size when the host serves an embedded logo over the API.

---

## bookclerk_plugin_manifest::logo::LOGO_EXTENSIONS

*Constant*: `&[&str]`

Allowed image extensions for embedded logos (lowercase, including the dot).

Used by [`validate_logo`](logo.md#bookclerk_plugin_manifestlogovalidate_logo) when classifying a relative path. Remote URLs are
not restricted to these extensions.



## bookclerk_plugin_manifest::logo::LogoKind

*Enum*

Classified `plugin.toml` `logo` value after successful validation.

Produced by [`validate_logo`](logo.md#bookclerk_plugin_manifestlogovalidate_logo) / `PluginManifest::logo_kind`.

**Variants:**
- `RemoteUrl(String)` - Browser-loadable `http://` or `https://` URL (no userinfo).
- `EmbeddedPath(String)` - Path relative to the plugin install root (forward-slash normalized).

**Methods:**

- `fn as_str(self: &Self) -> &str` - Returns the underlying URL or relative path string.
- `fn is_remote(self: &Self) -> bool` - Returns `true` when this logo is a remote `http(s)` URL.
- `fn is_embedded(self: &Self) -> bool` - Returns `true` when this logo is an embedded path under the plugin root.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> LogoKind`
- **PartialEq**
  - `fn eq(self: &Self, other: &LogoKind) -> bool`



## bookclerk_plugin_manifest::logo::MAX_EMBEDDED_LOGO_BYTES

*Constant*: `u64`

Maximum byte size when the host serves an embedded logo over the API.

Enforcement is host-side at read time; this constant documents the shared
budget (`512 KiB`).



## bookclerk_plugin_manifest::logo::embedded_logo_api_path

*Function*

Builds the Settings / UI href for an embedded logo.

Format: `/api/plugins/{kind}/{id}/logo`. Remote logos are not rewritten;
callers should use the remote URL string directly.

# Arguments

* `kind` - Plugin kind wire name (`source`, `integration`, `output`, `database`).
* `id` - Validated plugin id.

# Returns

Absolute path (no host) for the operator SPA or daemon static route.

```rust
fn embedded_logo_api_path(kind: &str, id: &str) -> String
```



## bookclerk_plugin_manifest::logo::logo_content_type

*Function*

Returns the MIME type for an embedded logo path based on its extension.

Unrecognized extensions map to `application/octet-stream`. Case of the
path is ignored.

# Arguments

* `path` - Relative logo path (typically from `LogoKind::EmbeddedPath`).

# Returns

A static MIME type string suitable for `Content-Type` when serving the file.

```rust
fn logo_content_type(path: &str) -> &'static str
```



## bookclerk_plugin_manifest::logo::validate_logo

*Function*

Validates an optional logo string from `plugin.toml`.

Absolute URLs (any scheme) are parsed with `url::Url`; only `http` /
`https` are accepted, and userinfo is forbidden. Values that fail URL
parse are treated as relative embedded image paths (must stay under the
plugin root and end with an allowed extension).

# Arguments

* `raw` - Raw `logo` field value (leading/trailing whitespace is trimmed).

# Returns

A classified [`LogoKind`](logo.md#bookclerk_plugin_manifestlogologokind).

# Errors

Returns `Error::Message` when the value is empty, contains NUL, uses a
disallowed URL scheme, includes userinfo, escapes the plugin root (`..`,
absolute / drive / UNC paths), or lacks an allowed image extension.

# Examples

```
use bookclerk_plugin_manifest::{validate_logo, LogoKind};

assert!(matches!(
    validate_logo("https://example.com/logo.png").unwrap(),
    LogoKind::RemoteUrl(_)
));
assert_eq!(
    validate_logo("assets/logo.png").unwrap(),
    LogoKind::EmbeddedPath("assets/logo.png".into())
);
assert!(validate_logo("javascript:alert(1)").is_err());
```

```rust
fn validate_logo(raw: &str) -> crate::error::Result<LogoKind>
```



