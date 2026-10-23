**bookclerk_plugin_manifest > error**

# Module: error

## Contents

**Enums**

- [`Error`](#error) - Errors produced while deserializing or validating `plugin.toml`.

**Type Aliases**

- [`Result`](#result) - Result alias for manifest parse, validate, and format operations.

---

## bookclerk_plugin_manifest::error::Error

*Enum*

Errors produced while deserializing or validating `plugin.toml`.

Display implementations are operator-facing (prefixed where useful, e.g.
`toml: …`). There is no structured error code; callers typically match on
the variant or format the message for CLI / doctor output.

**Variants:**
- `Message(String)` - Operator-facing semantic failure with no structured code.
- `TomlDe(toml::de::Error)` - TOML deserialize failure from `toml::from_str`.
- `TomlSer(toml::ser::Error)` - TOML serialize failure from canonical [`crate::format_manifest`](fmt.md#bookclerk_plugin_manifestfmtformat_manifest).

**Methods:**

- `fn message<impl Into<String>>(msg: impl Trait) -> Self` - Builds an `Error::Message` from any string-like value.

**Trait Implementations:**

- **From**
  - `fn from(source: toml::ser::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(source: toml::de::Error) -> Self`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`



## bookclerk_plugin_manifest::error::Result

*Type Alias*: `std::result::Result<T, Error>`

Result alias for manifest parse, validate, and format operations.



