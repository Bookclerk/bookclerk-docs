# bookclerk_workerd::granted_webhooks

`POST /webhooks/<op>` on the granted reverse channel: a workerd isolate's
`env.WEBHOOKS` calls, parsed and answered over plain byte slices.

The isolate holds a webhooks-only grant token minted at `open`; each call
names its operation in the path (`url`, `pull`, `ack`) and carries that
operation's fields as JSON. Every request is checked with the same
`bookclerk_plugin_abi::webhooks` rules the host applies again when it
serves `Webhooks`, so a malformed call fails here with a typed error and
never reaches the host. Keeping parsing sans-IO lets the `webhook_route`
fuzz target drive the code the bridge runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`RouteUrl`](#routeurl) | struct | `POST /webhooks/url` body. |
| [`RoutePull`](#routepull) | struct | `POST /webhooks/pull` body. |
| [`RouteAck`](#routeack) | struct | `POST /webhooks/ack` body. |
| [`WebhookCall`](#webhookcall) | enum | One checked `env.WEBHOOKS` call, ready for the host's `Webhooks` stub. |
| [`WebhookReply`](#webhookreply) | enum | What the host answered one `WebhookCall` with. |
| [`parse_webhook_call`](#parse_webhook_call) | function | Parses and checks one `POST /webhooks/<op>` call. |
| [`webhook_reply_json`](#webhook_reply_json) | function | Encodes a host answer as the route's JSON reply body. |
| [`MAX_WEBHOOK_ROUTE_BODY_BYTES`](#max_webhook_route_body_bytes) | constant | Largest `/webhooks/*` request body: an ack naming the most leases in both lists at their longest, with JSON quoting, plus room for the name. The ABI checks bound each field once decoded; this only bounds the read. |
| [`WEBHOOK_ROUTE_OPS`](#webhook_route_ops) | constant | Operations the route serves, in path spelling. |

## Structs

### RouteUrl

```rust
pub struct RouteUrl {
    pub name: String,
}
```

`POST /webhooks/url` body.

**Fields**

- `name`: `[[webhooks]]` name.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RoutePull

```rust
pub struct RoutePull {
    pub name: String,
    pub batch_size: u32,
    pub visibility_timeout_ms: u32,
}
```

`POST /webhooks/pull` body.

**Fields**

- `name`: `[[webhooks]]` name.
- `batch_size`: Most deliveries to lease, 1 to `maxWebhookPullBatch`.
- `visibility_timeout_ms`: Lease length in milliseconds; zero or absent means one minute.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RouteAck

```rust
pub struct RouteAck {
    pub name: String,
    pub acks: Vec<String>,
    pub retries: Vec<String>,
}
```

`POST /webhooks/ack` body.

**Fields**

- `name`: `[[webhooks]]` name.
- `acks`: Leases whose deliveries are done.
- `retries`: Leases whose deliveries the next pull offers again.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### WebhookCall

```rust
pub enum WebhookCall {
    Url(String),
    Pull(WebhookPullRequest),
    Ack(WebhookAckRequest),
}
```

One checked `env.WEBHOOKS` call, ready for the host's `Webhooks` stub.

**Variants**

- `Url`: `Webhooks.url`.
- `Pull`: `Webhooks.pull`.
- `Ack`: `Webhooks.ack`.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### WebhookReply

```rust
pub enum WebhookReply {
    Url(Option<String>),
    Pulled(WebhookPullPage),
    Done,
}
```

What the host answered one [`WebhookCall`](#webhookcall) with.

**Variants**

- `Url`: `url` answered this URL, or none.
- `Pulled`: `pull` leased these deliveries.
- `Done`: `ack` finished.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Functions

### parse_webhook_call

```rust
pub fn parse_webhook_call(op: &str, body: &[u8]) -> Result<WebhookCall, PluginError>
```

Parses and checks one `POST /webhooks/<op>` call.

#### Arguments

* `op` - Path segment after `/webhooks/`.
* `body` - Request body bytes, already bounded by
  [`MAX_WEBHOOK_ROUTE_BODY_BYTES`](#max_webhook_route_body_bytes).

#### Returns

The call, with every field within the ABI limits.

#### Errors

Returns `not_found` for an operation the route does not serve, and
`invalid_params` for malformed JSON, an unknown field, a malformed name
or lease id, or a batch size or lease length out of range.

### webhook_reply_json

```rust
pub fn webhook_reply_json(reply: &WebhookReply) -> Result<Vec<u8>>
```

Encodes a host answer as the route's JSON reply body.

`url` answers `{"url": "..."}` (or `null`), `pull` answers the
[`WebhookPullPage`](../bookclerk_plugin_abi/generated.md#webhookpullpage) (bodies as base64), and `ack` answers `{}`.

#### Errors

Returns the serializer's error, which a well-formed page never produces.

## Constants

### MAX_WEBHOOK_ROUTE_BODY_BYTES

```rust
pub const MAX_WEBHOOK_ROUTE_BODY_BYTES: u32 = _;
```

Largest `/webhooks/*` request body: an ack naming the most leases in
both lists at their longest, with JSON quoting, plus room for the name.
The ABI checks bound each field once decoded; this only bounds the read.

### WEBHOOK_ROUTE_OPS

```rust
pub const WEBHOOK_ROUTE_OPS: [&str; 3] = _;
```

Operations the route serves, in path spelling.
