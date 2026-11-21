# bookclerk_plugin_abi::network_grant

The network grant a guest sees at `open` (`Bindings.network`).

Read-only feedback: the host enforces the grant whatever the guest does
with it. A guest uses it to tell an unapproved destination (a network
setting whose value awaits the operator's consent) from one that is
simply unreachable, and to say so to the operator.

| Item | Kind | Summary |
| --- | --- | --- |
| [`SettingNetworkGrant`](#settingnetworkgrant) | struct | One network setting and the destination its value names. |
| [`NetworkGrant`](#networkgrant) | struct | The network grant in effect for one `PluginWorker.open`. |
| [`SettingGrantState`](#settinggrantstate) | enum | Where one network setting's destination stands. |
| [`MAX_NETWORK_GRANT_ENTRIES`](#max_network_grant_entries) | constant | Most entries in each list of a `NetworkGrant` (schema `maxNetworkGrantEntries`). |

## Structs

### SettingNetworkGrant

```rust
pub struct SettingNetworkGrant {
    pub key: String,
    pub network: SettingNetwork,
    pub destination: String,
    pub state: SettingGrantState,
    pub error: String,
}
```

One network setting and the destination its value names.

**Fields**

- `key`: The setting key.
- `network`: Whether the plugin fetches the destination or opens a socket to it.
- `destination`: The destination as the consent form shows it: `scheme://host:port`
  for a fetch, `host:port` for TCP (IPv6 in brackets); empty when
  [`Self::state`](#settingnetworkgrant) is unset or invalid.
- `state`: Where the destination stands.
- `error`: Why the value names no destination, when [`Self::state`](#settingnetworkgrant) is invalid;
  empty otherwise.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NetworkGrant

```rust
pub struct NetworkGrant {
    pub outbound: bool,
    pub domains: Vec<String>,
    pub tcp: Vec<String>,
    pub settings: Vec<SettingNetworkGrant>,
}
```

The network grant in effect for one `PluginWorker.open`.

**Fields**

- `outbound`: True when the grant allows outbound connections; false for `deny`.
- `domains`: Fetch destinations in effect, as `plugin.toml` `domains` entries,
  including approved setting destinations.
- `tcp`: Raw TCP destinations in effect, as `host:port`, one per granted port.
- `settings`: One entry per network setting, in declaration order.

#### NetworkGrant::setting_for

```rust
pub fn setting_for(&self, destination: &str) -> Option<&SettingNetworkGrant>
```

The network setting whose value names `destination` (as
[`SettingNetworkGrant::destination`](#settingnetworkgrant) spells it), if one does.

#### NetworkGrant::truncate_to_bound

```rust
pub fn truncate_to_bound(&mut self)
```

Keeps at most [`MAX_NETWORK_GRANT_ENTRIES`](#max_network_grant_entries) entries in each list, so
the grant always fits the schema's bound.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### SettingGrantState

```rust
pub enum SettingGrantState {
    Approved,
    Pending,
    Unset,
    Invalid,
}
```

Where one network setting's destination stands.

**Variants**

- `Approved`: The value names a destination the operator approved for this setting;
  the plugin reaches it.
- `Pending`: The value names a destination the operator has not approved yet; the
  plugin cannot reach it until the consent form is answered.
- `Unset`: The setting is empty or unset: it names no destination.
- `Invalid`: The value does not name a usable destination.

#### SettingGrantState::as_str

```rust
pub fn as_str(self) -> &'static str
```

Wire name (`approved`, `pending`, `unset`, `invalid`).

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

## Constants

### MAX_NETWORK_GRANT_ENTRIES

```rust
pub const MAX_NETWORK_GRANT_ENTRIES: usize = 256;
```

Most entries in each list of a [`NetworkGrant`](#networkgrant) (schema
`maxNetworkGrantEntries`).
