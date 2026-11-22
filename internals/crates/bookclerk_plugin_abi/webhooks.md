# bookclerk_plugin_abi::webhooks

Sans-IO checks for host-served inbound webhooks (`Bindings.webhooks`).

The guest SDK applies them before it calls, the host when it serves
`Webhooks`, and both sides before a delivery crosses the wire. Keeping
them over plain values lets the fuzz target and the unit tests drive the
code the host runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WebhookPull`](#webhookpull) | struct | A pull request after its checks, with the visibility timeout resolved. |
| [`is_valid_webhook_name`](#is_valid_webhook_name) | function | True when `name` is a valid webhook name: `a-z0-9*`, at most `maxWebhookNameBytes` bytes. |
| [`is_valid_webhook_header_name`](#is_valid_webhook_header_name) | function | True when `name` is a lowercase HTTP header name: RFC 9110 token characters, lowercase, at most `MAX_WEBHOOK_HEADER_NAME_BYTES` bytes. |
| [`check_webhook_name`](#check_webhook_name) | function | Fails closed on a malformed webhook name. |
| [`check_webhook_lease_id`](#check_webhook_lease_id) | function | Fails closed on a malformed lease id: 1 to `maxWebhookLeaseIdBytes` bytes of ASCII letters, digits, `-` and `_`. |
| [`check_webhook_pull`](#check_webhook_pull) | function | Checks a `Webhooks.pull` request and resolves its visibility timeout. |
| [`check_webhook_ack`](#check_webhook_ack) | function | Checks a `Webhooks.ack` request. |
| [`check_webhook_delivery`](#check_webhook_delivery) | function | Checks one delivery the host is about to hand a guest (or a guest received). |
| [`check_webhook_page`](#check_webhook_page) | function | Checks a pull page against the request it answers. |
| [`check_webhook_delivered`](#check_webhook_delivered) | function | Checks a `WebhookHandler.delivered` event. |
| [`MAX_WEBHOOK_HEADER_NAME_BYTES`](#max_webhook_header_name_bytes) | constant | Longest header name a delivery may carry, in bytes. |
| [`DEFAULT_WEBHOOK_VISIBILITY_MS`](#default_webhook_visibility_ms) | constant | Visibility timeout a pull uses when the guest passes zero, in milliseconds (one minute). |

## Structs

### WebhookPull

```rust
pub struct WebhookPull {
    pub batch_size: u32,
    pub visibility_ms: u32,
}
```

A pull request after its checks, with the visibility timeout resolved.

**Fields**

- `batch_size`: Most deliveries to lease, 1 to `maxWebhookPullBatch`.
- `visibility_ms`: How long leased deliveries stay hidden, in milliseconds, 1 to
  `maxWebhookVisibilityMs`.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### is_valid_webhook_name

```rust
pub fn is_valid_webhook_name(name: &str) -> bool
```

True when `name` is a valid webhook name: `[a-z0-9][a-z0-9-]*`, at most
`maxWebhookNameBytes` bytes.

### is_valid_webhook_header_name

```rust
pub fn is_valid_webhook_header_name(name: &str) -> bool
```

True when `name` is a lowercase HTTP header name: RFC 9110 token
characters, lowercase, at most [`MAX_WEBHOOK_HEADER_NAME_BYTES`](#max_webhook_header_name_bytes) bytes.

### check_webhook_name

```rust
pub fn check_webhook_name(name: &str) -> Result<()>
```

Fails closed on a malformed webhook name.

#### Errors

Returns `invalid_params` when `name` is not a valid webhook name.

### check_webhook_lease_id

```rust
pub fn check_webhook_lease_id(lease: &str) -> Result<()>
```

Fails closed on a malformed lease id: 1 to `maxWebhookLeaseIdBytes`
bytes of ASCII letters, digits, `-` and `_`.

#### Errors

Returns `invalid_params` for a malformed lease id.

### check_webhook_pull

```rust
pub fn check_webhook_pull(request: &WebhookPullRequest) -> Result<WebhookPull>
```

Checks a `Webhooks.pull` request and resolves its visibility timeout.

#### Arguments

* `request` - The guest's request.

#### Returns

The batch size and the visibility timeout to use.

#### Errors

Returns `invalid_params` for a malformed name, a batch size outside 1 to
`maxWebhookPullBatch`, or a visibility timeout above
`maxWebhookVisibilityMs`.

### check_webhook_ack

```rust
pub fn check_webhook_ack(request: &WebhookAckRequest) -> Result<()>
```

Checks a `Webhooks.ack` request.

#### Errors

Returns `invalid_params` for a malformed name or lease id, more than
`maxWebhookPullBatch` leases in either list, or a lease named in both.

### check_webhook_delivery

```rust
pub fn check_webhook_delivery(delivery: &WebhookDelivery) -> Result<()>
```

Checks one delivery the host is about to hand a guest (or a guest
received).

#### Errors

Returns `invalid_params` for a malformed lease id or header, or more
than `maxWebhookHeaders + 1` headers (the row's own plus
`content-type`); `payload_too_large` for a body over
`maxWebhookBodyBytes`.

### check_webhook_page

```rust
pub fn check_webhook_page(page: &WebhookPullPage, batch_size: u32) -> Result<()>
```

Checks a pull page against the request it answers.

#### Arguments

* `page` - The page the host returned.
* `batch_size` - The batch size the guest asked for.

#### Errors

Returns `invalid_params` when the page holds more deliveries than asked
for, or any delivery fails [`check_webhook_delivery`](#check_webhook_delivery).

### check_webhook_delivered

```rust
pub fn check_webhook_delivered(event: &WebhookDelivered) -> Result<()>
```

Checks a `WebhookHandler.delivered` event.

#### Errors

Returns `invalid_params` for a malformed name.

## Constants

### MAX_WEBHOOK_HEADER_NAME_BYTES

```rust
pub const MAX_WEBHOOK_HEADER_NAME_BYTES: usize = 64;
```

Longest header name a delivery may carry, in bytes.

### DEFAULT_WEBHOOK_VISIBILITY_MS

```rust
pub const DEFAULT_WEBHOOK_VISIBILITY_MS: u32 = 60_000;
```

Visibility timeout a pull uses when the guest passes zero, in
milliseconds (one minute).
