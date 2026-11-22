# bookclerk_plugin_manifest::webhooks

`[[webhooks]]`: inbound HTTP endpoints the host serves for a plugin.

The host serves each declared webhook at a secret per-instance URL under
the library's public origin, outside operator auth. Before anything is
stored it enforces the row's body cap, rate limit and signature rule; a
verified request is stored and answered at once, and the plugin reads
stored deliveries through its `WEBHOOKS` binding (see
`docs/adr/storage-destinations.md`, decision 6).

Every limit has a hard ceiling here, and the signature rule is required:
a plugin that takes unsigned requests says `scheme = "none"`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WebhookManifest`](#webhookmanifest) | struct | One `[[webhooks]]` row. |
| [`HmacSignature`](#hmacsignature) | struct | An HMAC-SHA256 signature rule. |
| [`WebhookSignature`](#webhooksignature) | enum | How the host checks that a request came from the sender. |
| [`SignatureEncoding`](#signatureencoding) | enum | How a signature digest is written in its header. |
| [`validate_webhooks`](#validate_webhooks) | function | Checks every `[[webhooks]]` row. |
| [`WEBHOOKS_BINDING`](#webhooks_binding) | constant | The binding name a plugin with `[[webhooks]]` receives; fixed, so no other binding may take it. |
| [`MAX_WEBHOOKS`](#max_webhooks) | constant | Most `[[webhooks]]` rows one plugin may declare. |
| [`MAX_WEBHOOK_NAME_LEN`](#max_webhook_name_len) | constant | Longest webhook name in bytes. |
| [`DEFAULT_WEBHOOK_MAX_BODY_BYTES`](#default_webhook_max_body_bytes) | constant | Default body cap in bytes (64 KiB). |
| [`MAX_WEBHOOK_BODY_BYTES`](#max_webhook_body_bytes) | constant | Largest body cap a row may declare, in bytes (1 MiB). |
| [`DEFAULT_WEBHOOK_RATE_PER_MINUTE`](#default_webhook_rate_per_minute) | constant | Default request rate per minute for one instance's webhook. |
| [`MAX_WEBHOOK_RATE_PER_MINUTE`](#max_webhook_rate_per_minute) | constant | Highest request rate per minute a row may declare. |
| [`DEFAULT_WEBHOOK_RETENTION_SECS`](#default_webhook_retention_secs) | constant | Default time a stored delivery waits for the plugin, in seconds (1 day). |
| [`MAX_WEBHOOK_RETENTION_SECS`](#max_webhook_retention_secs) | constant | Longest time a stored delivery may wait, in seconds (7 days). |
| [`MAX_WEBHOOK_HEADERS`](#max_webhook_headers) | constant | Most request headers a row may keep with each delivery. |
| [`MAX_WEBHOOK_HEADER_NAME_LEN`](#max_webhook_header_name_len) | constant | Longest header name a row may name, in bytes. |
| [`MAX_WEBHOOK_SIGNATURE_PREFIX_LEN`](#max_webhook_signature_prefix_len) | constant | Longest signature prefix, in bytes. |

## Re-exports

- [`bookclerk_plugin_abi::webhooks::is_valid_webhook_header_name` as `is_valid_header_name`](../bookclerk_plugin_abi/webhooks.md#is_valid_webhook_header_name)
- [`bookclerk_plugin_abi::webhooks::is_valid_webhook_name`](../bookclerk_plugin_abi/webhooks.md#is_valid_webhook_name)

## Structs

### WebhookManifest

```rust
pub struct WebhookManifest {
    pub name: String,
    pub max_body_bytes: u32,
    pub rate_per_minute: u32,
    pub retention_secs: u32,
    pub headers: Vec<String>,
    pub signature: WebhookSignature,
}
```

One `[[webhooks]]` row.

**Fields**

- `name`: URL path segment and the name the binding uses
  (`[a-z0-9][a-z0-9-]*`, at most [`MAX_WEBHOOK_NAME_LEN`](#max_webhook_name_len) bytes),
  unique within the plugin.
- `max_body_bytes`: Largest body the host accepts, in bytes (default
  [`DEFAULT_WEBHOOK_MAX_BODY_BYTES`](#default_webhook_max_body_bytes), at most
  [`MAX_WEBHOOK_BODY_BYTES`](#max_webhook_body_bytes)); a larger one is refused with 413.
- `rate_per_minute`: Requests per minute the host accepts for one instance (default
  [`DEFAULT_WEBHOOK_RATE_PER_MINUTE`](#default_webhook_rate_per_minute), at most
  [`MAX_WEBHOOK_RATE_PER_MINUTE`](#max_webhook_rate_per_minute)); more are refused with 429.
- `retention_secs`: Seconds a stored delivery waits for the plugin before it is dropped
  and counted as missed (default [`DEFAULT_WEBHOOK_RETENTION_SECS`](#default_webhook_retention_secs), at
  most [`MAX_WEBHOOK_RETENTION_SECS`](#max_webhook_retention_secs)).
- `headers`: Lowercase request header names kept with each delivery, besides
  `content-type`, which is always kept. Credential and hop headers are
  refused.
- `signature`: The check the host runs before storing a request.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### HmacSignature

```rust
pub struct HmacSignature {
    pub header: String,
    pub prefix: String,
    pub encoding: SignatureEncoding,
    pub secret_setting: String,
}
```

An HMAC-SHA256 signature rule.

**Fields**

- `header`: Lowercase header that carries the signature.
- `prefix`: Text before the digest in the header value (for example `v1=`);
  empty when the value is the digest alone.
- `encoding`: How the digest is written.
- `secret_setting`: Key of a `type = "secret"` `[[settings]]` entry holding the shared
  key. The host reads it; the plugin never needs it.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### WebhookSignature

```rust
pub enum WebhookSignature {
    None,
    HmacSha256(HmacSignature),
}
```

How the host checks that a request came from the sender.

**Variants**

- `None`: No check: every request within the caps is stored. The plugin must
  treat deliveries as untrusted hints.
- `HmacSha256`: HMAC-SHA256 of the raw body under a sealed plugin setting, carried in
  one header as `<prefix><digest>`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SignatureEncoding

```rust
pub enum SignatureEncoding {
    Hex,
    Base64,
}
```

How a signature digest is written in its header.

**Variants**

- `Hex`: Lowercase or uppercase hexadecimal.
- `Base64`: Standard base64 with padding.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### validate_webhooks

```rust
pub fn validate_webhooks(
    rows: &[WebhookManifest],
    is_secret_setting: impl Fn(&str) -> bool,
) -> Result<()>
```

Checks every `[[webhooks]]` row.

#### Arguments

* `rows` - the declared webhooks.
* `is_secret_setting` - whether a `[[settings]]` key names a
  `type = "secret"` entry.

#### Errors

Names the first row with a bad name, an out-of-range limit, a header the
host cannot keep, or a signature rule that does not resolve.

## Constants

### WEBHOOKS_BINDING

```rust
pub const WEBHOOKS_BINDING: &str = "WEBHOOKS";
```

The binding name a plugin with `[[webhooks]]` receives; fixed, so no
other binding may take it.

### MAX_WEBHOOKS

```rust
pub const MAX_WEBHOOKS: usize = 8;
```

Most `[[webhooks]]` rows one plugin may declare.

### MAX_WEBHOOK_NAME_LEN

```rust
pub const MAX_WEBHOOK_NAME_LEN: usize = _;
```

Longest webhook name in bytes.

### DEFAULT_WEBHOOK_MAX_BODY_BYTES

```rust
pub const DEFAULT_WEBHOOK_MAX_BODY_BYTES: u32 = _;
```

Default body cap in bytes (64 KiB).

### MAX_WEBHOOK_BODY_BYTES

```rust
pub const MAX_WEBHOOK_BODY_BYTES: u32 = bookclerk_plugin_abi::MAX_WEBHOOK_BODY_BYTES;
```

Largest body cap a row may declare, in bytes (1 MiB).

### DEFAULT_WEBHOOK_RATE_PER_MINUTE

```rust
pub const DEFAULT_WEBHOOK_RATE_PER_MINUTE: u32 = 60;
```

Default request rate per minute for one instance's webhook.

### MAX_WEBHOOK_RATE_PER_MINUTE

```rust
pub const MAX_WEBHOOK_RATE_PER_MINUTE: u32 = 600;
```

Highest request rate per minute a row may declare.

### DEFAULT_WEBHOOK_RETENTION_SECS

```rust
pub const DEFAULT_WEBHOOK_RETENTION_SECS: u32 = 86_400;
```

Default time a stored delivery waits for the plugin, in seconds (1 day).

### MAX_WEBHOOK_RETENTION_SECS

```rust
pub const MAX_WEBHOOK_RETENTION_SECS: u32 = _;
```

Longest time a stored delivery may wait, in seconds (7 days).

### MAX_WEBHOOK_HEADERS

```rust
pub const MAX_WEBHOOK_HEADERS: usize = _;
```

Most request headers a row may keep with each delivery.

### MAX_WEBHOOK_HEADER_NAME_LEN

```rust
pub const MAX_WEBHOOK_HEADER_NAME_LEN: usize = bookclerk_plugin_abi::webhooks::MAX_WEBHOOK_HEADER_NAME_BYTES;
```

Longest header name a row may name, in bytes.

### MAX_WEBHOOK_SIGNATURE_PREFIX_LEN

```rust
pub const MAX_WEBHOOK_SIGNATURE_PREFIX_LEN: usize = 16;
```

Longest signature prefix, in bytes.
