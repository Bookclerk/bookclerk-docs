# bookclerk_plugin_manifest::error

Manifest parse and semantic-validation errors.

All fallible entry points in this crate ([`crate::parse`](index.md#parse),
[`crate::validate_plugin_id`](id.md#validate_plugin_id), [`crate::validate_logo`](logo.md#validate_logo),
[`crate::format_manifest`](fmt.md#format_manifest), and so on) return [`Result`](#result) with this
[`enum@Error`](#error) type. TOML serde failures are preserved via
[`Error::TomlDe`](#error) / [`Error::TomlSer`](#error); semantic rules use
[`Error::Message`](#error).

| Item | Kind | Summary |
| --- | --- | --- |
| [`Error`](#error) | enum | Errors produced while deserializing or validating `plugin.toml`. |
| [`Result`](#result) | type alias | Result alias for manifest parse, validate, and format operations. |

## Enums

### Error

```rust
pub enum Error {
    Message(String),
    TomlDe(Error),
    TomlSer(Error),
}
```

Errors produced while deserializing or validating `plugin.toml`.

Display implementations are operator-facing (prefixed where useful, e.g.
`toml: …`). There is no structured error code; callers typically match on
the variant or format the message for CLI / doctor output.

**Variants**

- `Message`: Operator-facing semantic failure with no structured code.
  
  Used for id grammar, runtime field requirements, network domain rules,
  logo validation, and similar checks after TOML deserialize succeeds.
- `TomlDe`: TOML deserialize failure from `toml::from_str`.
- `TomlSer`: TOML serialize failure from canonical [`crate::format_manifest`](fmt.md#format_manifest).

#### Error::message

```rust
pub fn message(msg: impl Into<String>) -> Self
```

Builds an [`Error::Message`](#error) from any string-like value.

##### Arguments

* `msg` - Human-readable explanation (often prefixed with `plugin.toml:`).

##### Returns

An [`Error::Message`](#error) wrapping the owned string.

Implements: `Debug`, `Display`, `Error`, `From<Error>`

## Type aliases

### Result

```rust
pub type Result<T> = Result<T, Error>;
```

Result alias for manifest parse, validate, and format operations.
