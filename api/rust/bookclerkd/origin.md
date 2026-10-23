**bookclerkd > origin**

# Module: origin

## Contents

**Functions**

- [`configured_public_origin`](#configured_public_origin) - Trimmed configured origin with no trailing slash, or `None` when unset.
- [`detected_origin`](#detected_origin) - Origin detected from this request, rewritten to `localhost` on loopback IPs.
- [`effective_origin_from_config`](#effective_origin_from_config) - Effective origin from a live [`Config`](../bookclerk_config/settings.md#bookclerk_configsettingsconfig) plus optional request headers.
- [`effective_public_origin`](#effective_public_origin) - Effective public origin: pinned config, else bound loopback request, else localhost.
- [`loopback_fallback`](#loopback_fallback) - Loopback issuer when nothing is configured or detected.
- [`request_origin`](#request_origin) - Request `Origin` when it is a usable absolute origin (not `null`).
- [`rewrite_loopback_host`](#rewrite_loopback_host) - Rewrite loopback IPs (`127.0.0.1`, `::1`) to `localhost`, keeping scheme/port.

---

## bookclerkd::origin::configured_public_origin

*Function*

Trimmed configured origin with no trailing slash, or `None` when unset.

```rust
fn configured_public_origin(public_origin: Option<&str>) -> Option<String>
```



## bookclerkd::origin::detected_origin

*Function*

Origin detected from this request, rewritten to `localhost` on loopback IPs.

Ignores `Origin` / `Host` when any forwarding header is present, or when the
authority is not a bound loopback listen port.

```rust
fn detected_origin(headers: &axum::http::HeaderMap, listen: &bookclerk_config::ListenAddrs) -> String
```



## bookclerkd::origin::effective_origin_from_config

*Function*

Effective origin from a live [`Config`](../bookclerk_config/settings.md#bookclerk_configsettingsconfig) plus optional request headers.

```rust
fn effective_origin_from_config(cfg: &bookclerk_config::Config, headers: Option<&axum::http::HeaderMap>) -> String
```



## bookclerkd::origin::effective_public_origin

*Function*

Effective public origin: pinned config, else bound loopback request, else localhost.

```rust
fn effective_public_origin(public_origin: Option<&str>, headers: Option<&axum::http::HeaderMap>, listen: &bookclerk_config::ListenAddrs) -> String
```



## bookclerkd::origin::loopback_fallback

*Function*

Loopback issuer when nothing is configured or detected.

```rust
fn loopback_fallback(listen: &bookclerk_config::ListenAddrs) -> String
```



## bookclerkd::origin::request_origin

*Function*

Request `Origin` when it is a usable absolute origin (not `null`).

```rust
fn request_origin(headers: &axum::http::HeaderMap) -> Option<String>
```



## bookclerkd::origin::rewrite_loopback_host

*Function*

Rewrite loopback IPs (`127.0.0.1`, `::1`) to `localhost`, keeping scheme/port.

```rust
fn rewrite_loopback_host(origin: &str) -> String
```



