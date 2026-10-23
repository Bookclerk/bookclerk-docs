# bookclerk_plugin_manifest::address

Address-space policy independent of hostname matching.

Default reachability is the public Internet. Loopback, RFC1918, link-local,
unique-local, metadata, and IPv4-mapped forms of those ranges stay denied
unless the operator grants an explicit CIDR. A public-redirect capability
never implies those ranges.

| Item | Kind | Summary |
| --- | --- | --- |
| [`CidrGrant`](#cidrgrant) | struct | One operator- or manifest-granted CIDR (`10.0.60.100/32`, `::1/128`). |
| [`canonical_ip`](#canonical_ip) | function | Unwraps IPv4-mapped IPv6 so `:ffff:127.0.0.1` is treated as IPv4. |
| [`is_public_internet`](#is_public_internet) | function | True when `ip` is reachable on the public Internet (not a special range). |
| [`is_restricted_hostname`](#is_restricted_hostname) | function | Special hostnames that resolve into denied address space without a CIDR grant. |
| [`address_allowed`](#address_allowed) | function | True when `ip` is allowed: public Internet, or an explicit CIDR grant. |
| [`workerd_network_allow`](#workerd_network_allow) | function | Workerd `network.allow` tokens for this policy: `public` plus granted CIDRs. |

## Structs

### CidrGrant

```rust
pub struct CidrGrant {
    pub network: IpAddr,
    pub prefix: u8,
}
```

One operator- or manifest-granted CIDR (`10.0.60.100/32`, `::1/128`).

**Fields**

- `network`: Network address (host bits masked off).
- `prefix`: Prefix length.

#### CidrGrant::parse

```rust
pub fn parse(raw: &str) -> Result<Self, String>
```

Parses `addr/prefix`. Bare addresses are treated as `/32` or `/128`.

##### Errors

Returns a message when the CIDR is malformed or the prefix is too large.

#### CidrGrant::contains

```rust
pub fn contains(&self, ip: IpAddr) -> bool
```

True when `ip` (after IPv4-mapped unwrapping) falls in this range.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### canonical_ip

```rust
pub fn canonical_ip(ip: IpAddr) -> IpAddr
```

Unwraps IPv4-mapped IPv6 so `:ffff:127.0.0.1` is treated as IPv4.

### is_public_internet

```rust
pub fn is_public_internet(ip: IpAddr) -> bool
```

True when `ip` is reachable on the public Internet (not a special range).

### is_restricted_hostname

```rust
pub fn is_restricted_hostname(host: &str) -> bool
```

Special hostnames that resolve into denied address space without a CIDR grant.

### address_allowed

```rust
pub fn address_allowed(ip: IpAddr, grants: &[CidrGrant]) -> bool
```

True when `ip` is allowed: public Internet, or an explicit CIDR grant.

### workerd_network_allow

```rust
pub fn workerd_network_allow(cidrs: &[CidrGrant]) -> Vec<String>
```

Workerd `network.allow` tokens for this policy: `public` plus granted CIDRs.

Never emits the blanket `private` / `local` / `network` tokens.
