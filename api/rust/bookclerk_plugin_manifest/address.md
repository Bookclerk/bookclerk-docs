**bookclerk_plugin_manifest > address**

# Module: address

## Contents

**Structs**

- [`CidrGrant`](#cidrgrant) - One operator- or manifest-granted CIDR (`10.0.60.100/32`, `::1/128`).

**Functions**

- [`address_allowed`](#address_allowed) - True when `ip` is allowed: public Internet, or an explicit CIDR grant.
- [`canonical_ip`](#canonical_ip) - Unwraps IPv4-mapped IPv6 so `:ffff:127.0.0.1` is treated as IPv4.
- [`is_public_internet`](#is_public_internet) - True when `ip` is reachable on the public Internet (not a special range).
- [`is_restricted_hostname`](#is_restricted_hostname) - Special hostnames that resolve into denied address space without a CIDR grant.
- [`workerd_network_allow`](#workerd_network_allow) - Workerd `network.allow` tokens for this policy: `public` plus granted CIDRs.

---

## bookclerk_plugin_manifest::address::CidrGrant

*Struct*

One operator- or manifest-granted CIDR (`10.0.60.100/32`, `::1/128`).

**Fields:**
- `network: std::net::IpAddr` - Network address (host bits masked off).
- `prefix: u8` - Prefix length.

**Methods:**

- `fn parse(raw: &str) -> Result<Self, String>` - Parses `addr/prefix`. Bare addresses are treated as `/32` or `/128`.
- `fn contains(self: &Self, ip: IpAddr) -> bool` - True when `ip` (after IPv4-mapped unwrapping) falls in this range.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CidrGrant) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CidrGrant`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_manifest::address::address_allowed

*Function*

True when `ip` is allowed: public Internet, or an explicit CIDR grant.

```rust
fn address_allowed(ip: std::net::IpAddr, grants: &[CidrGrant]) -> bool
```



## bookclerk_plugin_manifest::address::canonical_ip

*Function*

Unwraps IPv4-mapped IPv6 so `:ffff:127.0.0.1` is treated as IPv4.

```rust
fn canonical_ip(ip: std::net::IpAddr) -> std::net::IpAddr
```



## bookclerk_plugin_manifest::address::is_public_internet

*Function*

True when `ip` is reachable on the public Internet (not a special range).

```rust
fn is_public_internet(ip: std::net::IpAddr) -> bool
```



## bookclerk_plugin_manifest::address::is_restricted_hostname

*Function*

Special hostnames that resolve into denied address space without a CIDR grant.

```rust
fn is_restricted_hostname(host: &str) -> bool
```



## bookclerk_plugin_manifest::address::workerd_network_allow

*Function*

Workerd `network.allow` tokens for this policy: `public` plus granted CIDRs.

Never emits the blanket `private` / `local` / `network` tokens.

```rust
fn workerd_network_allow(cidrs: &[CidrGrant]) -> Vec<String>
```



