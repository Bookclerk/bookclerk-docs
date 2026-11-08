# bookclerk_tls

The one TLS backend for the Bookclerk hosts.

Every first-party TLS client and server in the hosts uses rustls with the
aws-lc-rs crypto provider, which also brings the hybrid post-quantum key
exchange (`X25519MLKEM768`) rustls prefers. This crate is the single place
that names the provider, so no crate falls back to another one (ring or
OpenSSL) and the build compiles one crypto library.

- [`crypto_provider`](#crypto_provider) hands the provider to code that builds a rustls
  `ClientConfig` or `ServerConfig` itself (the node API listener and
  client).
- [`ureq`](ureq.md) is the `ureq` crate as host crates use it: `use bookclerk_tls::ureq;`
  and then `ureq::get(url)` as usual. Its request functions send through
  [`http_agent`](#http_agent) instead of `ureq`'s own default agent, which has no
  provider (the workspace builds `ureq` with `rustls-no-provider`) and would
  panic on its first HTTPS request. Host crates depend on this crate rather
  than on `ureq` directly, so that agent is out of reach.
- [`http_agent`](#http_agent) is the shared blocking HTTP client behind those functions.
- [`ureq_tls_config`](#ureq_tls_config) is the same TLS setting for crates that build a
  custom `ureq` agent.

See `docs/internals/tls.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`crypto_provider`](#crypto_provider) | function | The rustls crypto provider every host TLS connection uses: aws-lc-rs with its default cipher suites and key exchange groups. |
| [`ureq_tls_config`](#ureq_tls_config) | function | `ureq` TLS settings that use `crypto_provider` and Mozilla's root certificates (`webpki-roots`), as `ureq`'s default agent does. |
| [`http_agent`](#http_agent) | function | The shared blocking HTTP client for the hosts. |

## Modules

- [`ureq`](ureq.md): The `ureq` crate with the hosts' TLS: everything `ureq` exports, except that the request functions (`get`, `post`, …), `run` and `agent` use `http_agent`.

## Functions

### crypto_provider

```rust
pub fn crypto_provider() -> Arc<CryptoProvider>
```

The rustls crypto provider every host TLS connection uses: aws-lc-rs with
its default cipher suites and key exchange groups.

#### Returns

A shared handle to the provider. Every call returns the same instance, so
rustls and `ureq` can pool configurations built from it.

### ureq_tls_config

```rust
pub fn ureq_tls_config() -> TlsConfig
```

`ureq` TLS settings that use [`crypto_provider`](#crypto_provider) and Mozilla's root
certificates (`webpki-roots`), as `ureq`'s default agent does.

#### Returns

A TLS configuration to pass to `ureq::config::ConfigBuilder::tls_config`.

### http_agent

```rust
pub fn http_agent() -> Agent
```

The shared blocking HTTP client for the hosts.

It has `ureq`'s default settings apart from TLS, which uses
[`ureq_tls_config`](#ureq_tls_config). Callers that need other settings (timeouts, status
handling) set them per request, or build their own agent with
[`ureq_tls_config`](#ureq_tls_config).

#### Returns

A clone of one process-wide agent; clones share its connection pool.
