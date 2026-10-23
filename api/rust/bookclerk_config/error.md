**bookclerk_config > error**

# Module: error

## Contents

**Enums**

- [`ConfigError`](#configerror) - Errors produced while loading or validating configuration.

**Type Aliases**

- [`Result`](#result) - Result alias for configuration load and validation helpers in this crate.

---

## bookclerk_config::error::ConfigError

*Enum*

Errors produced while loading or validating configuration.

**Variants:**
- `Read{ path: String, source: std::io::Error }` - Failed to open or read a config file from disk.
- `Parse{ path: String, source: toml::de::Error }` - TOML syntax or type error while deserializing a config file.
- `Invalid(String)` - Semantic validation failure after a successful parse (missing required
- `Io(std::io::Error)` - Generic I/O failure unrelated to a specific config path context.

**Trait Implementations:**

- **From**
  - `fn from(source: std::io::Error) -> Self`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Error**
  - `fn source(self: &Self) -> ::core::option::Option<&dyn ::thiserror::__private21::Error>`



## bookclerk_config::error::Result

*Type Alias*: `std::result::Result<T, ConfigError>`

Result alias for configuration load and validation helpers in this crate.



