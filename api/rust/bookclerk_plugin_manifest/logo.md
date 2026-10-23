# bookclerk_plugin_manifest::logo

Plugin logo validation: remote `http(s)` URL or relative image path.

The optional `logo` key in `plugin.toml` is either a browser-loadable
remote URL (Settings / Accounts UI loads it directly) or a path under the
plugin install root that the host serves via
[`embedded_logo_api_path`](#embedded_logo_api_path). Absolute filesystem paths, parent segments,
and non-http(s) URL schemes are rejected.

| Item | Kind | Summary |
| --- | --- | --- |
| [`LogoKind`](#logokind) | enum | Classified `plugin.toml` `logo` value after successful validation. |
| [`validate_logo`](#validate_logo) | function | Validates an optional logo string from `plugin.toml`. |
| [`logo_content_type`](#logo_content_type) | function | Returns the MIME type for an embedded logo path based on its extension. |
| [`embedded_logo_api_path`](#embedded_logo_api_path) | function | Builds the Settings / UI href for an embedded logo. |
| [`LOGO_EXTENSIONS`](#logo_extensions) | constant | Allowed image extensions for embedded logos (lowercase, including the dot). |
| [`MAX_EMBEDDED_LOGO_BYTES`](#max_embedded_logo_bytes) | constant | Maximum byte size when the host serves an embedded logo over the API. |

## Enums

### LogoKind

```rust
pub enum LogoKind {
    RemoteUrl(String),
    EmbeddedPath(String),
}
```

Classified `plugin.toml` `logo` value after successful validation.

Produced by [`validate_logo`](#validate_logo) / [`PluginManifest::logo_kind`](types.md#pluginmanifestlogo_kind).

**Variants**

- `RemoteUrl`: Browser-loadable `http://` or `https://` URL (no userinfo).
  
  The string is the original trimmed input (scheme and path preserved).
- `EmbeddedPath`: Path relative to the plugin install root (forward-slash normalized).
  
  The host serves the file via [`embedded_logo_api_path`](#embedded_logo_api_path); must end with
  one of [`LOGO_EXTENSIONS`](#logo_extensions).

#### LogoKind::as_str

```rust
pub fn as_str(&self) -> &str
```

Returns the underlying URL or relative path string.

##### Returns

Borrowed contents of [`LogoKind::RemoteUrl`](#logokind) or [`LogoKind::EmbeddedPath`](#logokind).

#### LogoKind::is_remote

```rust
pub fn is_remote(&self) -> bool
```

Returns `true` when this logo is a remote `http(s)` URL.

#### LogoKind::is_embedded

```rust
pub fn is_embedded(&self) -> bool
```

Returns `true` when this logo is an embedded path under the plugin root.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### validate_logo

```rust
pub fn validate_logo(raw: &str) -> Result<LogoKind>
```

Validates an optional logo string from `plugin.toml`.

Absolute URLs (any scheme) are parsed with `url::Url`; only `http` /
`https` are accepted, and userinfo is forbidden. Values that fail URL
parse are treated as relative embedded image paths (must stay under the
plugin root and end with an allowed extension).

#### Arguments

* `raw` - Raw `logo` field value (leading/trailing whitespace is trimmed).

#### Returns

A classified [`LogoKind`](#logokind).

#### Errors

Returns [`Error::Message`](error.md#error) when the value is empty, contains NUL, uses a
disallowed URL scheme, includes userinfo, escapes the plugin root (`..`,
absolute / drive / UNC paths), or lacks an allowed image extension.

#### Examples

```rust
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

### logo_content_type

```rust
pub fn logo_content_type(path: &str) -> &'static str
```

Returns the MIME type for an embedded logo path based on its extension.

Unrecognized extensions map to `application/octet-stream`. Case of the
path is ignored.

#### Arguments

* `path` - Relative logo path (typically from [`LogoKind::EmbeddedPath`](#logokind)).

#### Returns

A static MIME type string suitable for `Content-Type` when serving the file.

### embedded_logo_api_path

```rust
pub fn embedded_logo_api_path(kind: &str, id: &str) -> String
```

Builds the Settings / UI href for an embedded logo.

Format: `/api/plugins/{kind}/{id}/logo`. Remote logos are not rewritten;
callers should use the remote URL string directly.

#### Arguments

* `kind` - Plugin kind wire name (`source`, `integration`, `output`, `database`).
* `id` - Validated plugin id.

#### Returns

Absolute path (no host) for the operator SPA or daemon static route.

## Constants

### LOGO_EXTENSIONS

```rust
pub const LOGO_EXTENSIONS: &[&str] = _;
```

Allowed image extensions for embedded logos (lowercase, including the dot).

Used by [`validate_logo`](#validate_logo) when classifying a relative path. Remote URLs are
not restricted to these extensions.

### MAX_EMBEDDED_LOGO_BYTES

```rust
pub const MAX_EMBEDDED_LOGO_BYTES: u64 = _;
```

Maximum byte size when the host serves an embedded logo over the API.

Enforcement is host-side at read time; this constant documents the shared
budget (`512 KiB`).
