**bookclerk_config > listen**

# Module: listen

## Contents

**Structs**

- [`ListenAddrs`](#listenaddrs) - One or more bind addresses for the HTTP control plane.

---

## bookclerk_config::listen::ListenAddrs

*Struct*

One or more bind addresses for the HTTP control plane.

TOML accepts either a string or an array; we always serialize as an array.

**Tuple Struct**: `()`

**Methods:**

- `fn new(addrs: Vec<String>) -> Self` - Build from raw address strings, dropping empties; falls back to defaults
- `fn parse_list(raw: &str) -> Result<Self, String>` - Parse a single address or a comma-separated list
- `fn as_slice(self: &Self) -> &[String]` - Configured bind strings in declaration order.
- `fn is_empty(self: &Self) -> bool` - True when no bind addresses remain (should not occur after `Self::new`).
- `fn join_comma(self: &Self) -> String` - Comma-joined form for settings API / overrides.
- `fn socket_addrs(self: &Self) -> Result<Vec<SocketAddr>, String>` - Parse each entry as a [`SocketAddr`](https://doc.rust-lang.org/stable/core/net/socket_addr/enum.SocketAddr.html).
- `fn validate(self: &Self) -> Result<(), String>` - Validate every entry parses as a socket address.
- `fn ui_port(self: &Self) -> u16` - Port used for tray `http://localhost:<port>` (first entry's port).
- `fn ports(self: &Self) -> Vec<u16>` - Distinct TCP ports from configured bind addresses.
- `fn has_loopback(self: &Self) -> bool` - Whether any configured bind is loopback.
- `fn has_unspecified(self: &Self) -> bool` - Whether any configured bind is unspecified (`0.0.0.0` / `::`).
- `fn tray_base_url(self: &Self) -> String` - Prefer `http://localhost:<port>` when any loopback or unspecified bind is

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<D>(deserializer: D) -> Result<Self, <D as >::Error>`
- **Serialize**
  - `fn serialize<S>(self: &Self, serializer: S) -> Result<<S as >::Ok, <S as >::Error>`
- **Display**
  - `fn fmt(self: &Self, f: & mut fmt::Formatter) -> fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ListenAddrs`
- **PartialEq**
  - `fn eq(self: &Self, other: &ListenAddrs) -> bool`
- **From**
  - `fn from(value: &str) -> Self`
- **Default**
  - `fn default() -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **FromStr**
  - `fn from_str(s: &str) -> Result<Self, <Self as >::Err>`



