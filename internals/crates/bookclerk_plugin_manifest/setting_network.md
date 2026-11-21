# bookclerk_plugin_manifest::setting_network

Network destinations named by plugin settings.

A `[[settings]]` entry may declare `network = "fetch"` (its value is a URL)
or `network = "tcp"` (its value is `host:port`). The host turns the value
an operator saved into one [`SettingDestination`](#settingdestination) with
[`derive_setting_destination`](#derive_setting_destination), lists it on the consent form with the
setting as its source, and adds it to the plugin's egress policy only once
the operator approved that exact destination.

The value is untrusted text (an operator typed it, or a plugin's own
default supplied it), so derivation is a pure function over the string
and is fuzzed (`fuzz/fuzz_targets/setting_destination.rs`).

| Item | Kind | Summary |
| --- | --- | --- |
| [`SettingDestination`](#settingdestination) | struct | The one network destination a setting's value names. |
| [`DestinationScope`](#destinationscope) | enum | What kind of address a setting's host is, which decides how the grant reaches it. |
| [`SettingDestinationError`](#settingdestinationerror) | enum | Why a setting value names no usable destination. |
| [`derive_setting_destination`](#derive_setting_destination) | function | Derives the destination a network setting's value names. |

## Structs

### SettingDestination

```rust
pub struct SettingDestination {
    pub network: SettingNetwork,
    pub host: String,
    pub port: u16,
    pub tls: bool,
    pub scope: DestinationScope,
}
```

The one network destination a setting's value names.

**Fields**

- `network`: Whether the plugin fetches it or opens a raw socket to it.
- `host`: The host: an IDNA-normalized name, or a canonical IP literal (IPv6
  without brackets, IPv4-mapped IPv6 unwrapped).
- `port`: The port, 1-65535: the value's own, else the setting's
  `default_port`, else the URL scheme's.
- `tls`: True for an `https` fetch, whose certificate is verified; false for
  plain HTTP and raw TCP, which trust the resolver.
- `scope`: What kind of address [`Self::host`](#settingdestination) is.

#### SettingDestination::domain_entry

```rust
pub fn domain_entry(&self) -> Option<String>
```

The `domains` entry a fetch destination adds (`host:port`), or `None`
for a TCP destination.

#### SettingDestination::tcp_grant

```rust
pub fn tcp_grant(&self) -> Option<TcpGrant>
```

The TCP grant a TCP destination adds, or `None` for a fetch.

#### SettingDestination::local_name

```rust
pub fn local_name(&self) -> Option<String>
```

The [`crate::EgressPolicy::local_names`](egress.md#egresspolicy) entry this destination adds:
its name, or its local-network address. `None` for a public or
restricted address.

#### SettingDestination::display

```rust
pub fn display(&self) -> String
```

How the destination reads on the consent form and in logs:
`https://abs.home.lan:13378` for a fetch, `db.home.lan:5432` for TCP.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

## Enums

### DestinationScope

```rust
pub enum DestinationScope {
    Name,
    PublicAddress,
    LocalAddress,
    Restricted,
}
```

What kind of address a setting's host is, which decides how the grant
reaches it.

**Variants**

- `Name`: A host name. It may resolve to a public address, or to a local-network
  address (RFC 1918, shared CGNAT space, IPv6 ULA) for this name only.
- `PublicAddress`: A public IP literal.
- `LocalAddress`: A local-network IP literal; the grant reaches exactly that address.
- `Restricted`: Loopback, link-local (cloud metadata), another special address, or a
  restricted name such as `localhost`. The destination is listed, but the
  connection still needs an address grant the operator adds explicitly.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### SettingDestinationError

```rust
pub enum SettingDestinationError {
    Empty,
    Malformed,
    Scheme,
    Credentials,
    Host,
    Port,
}
```

Why a setting value names no usable destination.

**Variants**

- `Empty`: The value is empty: the setting names no destination.
- `Malformed`: Longer than a setting value may be, or holds whitespace or control
  characters.
- `Scheme`: A `fetch` URL whose scheme is not `http` or `https`.
- `Credentials`: The value carries a user name or password; credentials belong in a
  secret setting.
- `Host`: The host is missing, a wildcard, percent-encoded or not a valid name.
- `Port`: The port is missing (a bare `tcp` host without `default_port`), zero
  or not a number.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### derive_setting_destination

```rust
pub fn derive_setting_destination(
    network: SettingNetwork,
    default_port: Option<u16>,
    value: &str,
) -> Result<SettingDestination, SettingDestinationError>
```

Derives the destination a network setting's value names.

- `fetch`: an `http://` or `https://` URL; a value without `://` is read
  as `https://<value>`. The port is the URL's own, else `default_port`
  for a bare value, else the scheme's (80 or 443). Paths and queries are
  ignored.
- `tcp`: `host:port`, `[v6]:port`, a bare host (port from
  `default_port`), or a URL of any scheme (its host and port, else
  `default_port`), so a `postgres://db:5432/x` value works.

A user name or password anywhere is refused, and so is a wildcard host.

#### Arguments

* `network` - The setting's declared [`SettingNetwork`](../bookclerk_plugin_abi/settings_schema.md#settingnetwork).
* `default_port` - The setting's declared `default_port`.
* `value` - The saved value (surrounding whitespace is ignored).

#### Returns

The normalized destination.

#### Errors

[`SettingDestinationError`](#settingdestinationerror) naming the first problem.

#### Examples

```rust
use bookclerk_plugin_abi::SettingNetwork;
use bookclerk_plugin_manifest::{derive_setting_destination, DestinationScope};

let abs = derive_setting_destination(SettingNetwork::Fetch, None, "http://ABS.home.lan:13378/")
    .unwrap();
assert_eq!(abs.domain_entry().as_deref(), Some("abs.home.lan:13378"));
assert_eq!(abs.scope, DestinationScope::Name);

let db = derive_setting_destination(SettingNetwork::Tcp, Some(5432), "192.168.1.5").unwrap();
assert_eq!(db.display(), "192.168.1.5:5432");
assert_eq!(db.scope, DestinationScope::LocalAddress);
```
