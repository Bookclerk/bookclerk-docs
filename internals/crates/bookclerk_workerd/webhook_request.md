# bookclerk_workerd::webhook_request

Sans-IO checks for one request on the public webhook route
(`POST /hooks/<token>/<name>`), in the order the route applies them.

The route is outside operator auth, so everything here reads bytes from
anyone on the Internet. It runs over plain values: the method, the
declared length, the raw header pairs and the body. The `webhook_request`
fuzz target and the unit tests drive the same functions the daemon does.
It lives beside the isolate routes so the fuzz build needs no host crate.

1. [`check_webhook_head`]: the method must be `POST` and a declared
   length must fit the row's `max_body_bytes`, before the body is read.
2. [`WebhookRateLimiter`]: at most the row's `rate_per_minute` requests
   per instance and webhook, as a token bucket.
3. [`verify_webhook_request`]: the body fits, the signature rule holds
   (HMAC-SHA256 compared in constant time), and the kept headers are
   picked out and checked.

| Item | Kind | Summary |
| --- | --- | --- |
| [`WebhookRateLimiter`](#webhookratelimiter) | struct | Token-bucket rate limit per instance and webhook, kept in memory on the node that serves the route. |
| [`WebhookRefusal`](#webhookrefusal) | enum | Why the route refused a request. Each maps onto one HTTP status, and the reply body never says more than the status does. |
| [`check_webhook_head`](#check_webhook_head) | function | Checks the request line and declared length before the body is read. |
| [`verify_webhook_request`](#verify_webhook_request) | function | Verifies a request whose body has been read, and picks out the headers the delivery keeps. |

## Structs

### WebhookRateLimiter

```rust
pub struct WebhookRateLimiter {
    /* private fields */
}
```

Token-bucket rate limit per instance and webhook, kept in memory on the
node that serves the route.

Each bucket holds up to the row's `rate_per_minute` tokens and refills
continuously at that rate; a request takes one. Buckets idle long enough
to be full again are dropped when the map grows past
[`WebhookRateLimiter::MAX_BUCKETS`](#webhookratelimitermax_buckets).

#### WebhookRateLimiter::MAX_BUCKETS

```rust
pub const MAX_BUCKETS: usize = 4096
```

Buckets kept before full, idle ones are swept.

#### WebhookRateLimiter::take

```rust
pub fn take(
    &mut self,
    instance: &str,
    name: &str,
    rate_per_minute: u32,
    now_ms: i64,
) -> Result<(), WebhookRefusal>
```

Takes one request from the bucket of `(instance, name)`.

##### Arguments

* `instance` - Plugin instance id.
* `name` - Webhook name.
* `rate_per_minute` - The row's rate, at least 1.
* `now_ms` - Current time in Unix milliseconds.

##### Errors

[`WebhookRefusal::RateLimited`](#webhookrefusal) when the bucket is empty.

Implements: `Debug`, `Default`

## Enums

### WebhookRefusal

```rust
pub enum WebhookRefusal {
    MethodNotAllowed,
    NotFound,
    RateLimited,
    PayloadTooLarge,
    BadRequest,
    BadSignature,
    Unavailable,
}
```

Why the route refused a request. Each maps onto one HTTP status, and
the reply body never says more than the status does.

**Variants**

- `MethodNotAllowed`: The method is not `POST` (405).
- `NotFound`: The token or name matches no webhook (404).
- `RateLimited`: The instance's webhook is over its request rate (429).
- `PayloadTooLarge`: The body is larger than the row allows (413).
- `BadRequest`: A header is malformed or repeated where it must appear once (400).
- `BadSignature`: The signature is missing or does not match (401).
- `Unavailable`: The webhook cannot take requests now: its signing secret is not set,
  webhooks are turned off, or the instance's waiting cap is full (503).

#### WebhookRefusal::status

```rust
pub fn status(self) -> u16
```

The HTTP status the route answers with.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

## Functions

### check_webhook_head

```rust
pub fn check_webhook_head(
    row: &WebhookManifest,
    method: &str,
    content_length: Option<u64>,
) -> Result<(), WebhookRefusal>
```

Checks the request line and declared length before the body is read.

#### Arguments

* `row` - The webhook's manifest row.
* `method` - Request method as received.
* `content_length` - The declared `Content-Length`, when one was sent.

#### Errors

[`WebhookRefusal::MethodNotAllowed`](#webhookrefusal) for anything but `POST`;
[`WebhookRefusal::PayloadTooLarge`](#webhookrefusal) when the declared length is over the
row's `max_body_bytes`.

### verify_webhook_request

```rust
pub fn verify_webhook_request(
    row: &WebhookManifest,
    secret: Option<&[u8]>,
    headers: &[(String, Vec<u8>)],
    body: &[u8],
) -> Result<Vec<WebhookHeader>, WebhookRefusal>
```

Verifies a request whose body has been read, and picks out the headers
the delivery keeps.

#### Arguments

* `row` - The webhook's manifest row.
* `secret` - The signing secret the row's rule names, when it has one
  and the operator set it.
* `headers` - Raw request headers as `(name, value)` pairs.
* `body` - The request body.

#### Returns

The kept headers, lowercase: `content-type` first when sent, then the
row's `headers` in the order it lists them. Absent headers are skipped.

#### Errors

[`WebhookRefusal::PayloadTooLarge`](#webhookrefusal) for a body over `max_body_bytes`;
the `check_hmac` refusals for a signed row; and
[`WebhookRefusal::BadRequest`](#webhookrefusal) for a kept header that repeats, is over
the value cap or holds a control character.
