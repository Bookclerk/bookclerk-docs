# bookclerk_workerd::invoke

`POST /invoke`: the Cap'n Proto data plane between the launcher and the
adapter isolate.

[`InvokeClient`](#invokeclient) is a Cap'n Proto `ClientHook`. Every method call made
through a typed client built from it (`ContentSourceClient`,
`EventConsumerClient`, `JobRunnerClient`, `DatabaseClient`, …) becomes one
HTTP request whose body is the `$Params` struct as an unpacked
single-segment message and whose `200` reply body is the `$Results` struct
— the same envelopes the generated `generated-wire.ts` codecs read and
write in the isolate. No JSON projection of any ABI struct exists on this
path; see `docs/workerd-bridge.md`.

Unpacked messages carry no capability table, so interface-typed fields are
written as capability indexes and the table travels beside the message as
the `X-Bookclerk-Caps` JSON header. Host-served capabilities are
[`GrantCap`](#grantcap)s — grant tokens the isolate presents on the granted channel —
and the only isolate-served kind is an `AdapterDatabaseSession`, which the
reply names by object id and later calls address with `X-Bookclerk-Target`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`GrantCap`](#grantcap) | struct | A capability that exists only to be described: it is never callable from this process. Placed in a `$Params` capability table it is serialized as its `CapDescriptor` in `X-Bookclerk-Caps`, and the isolate reaches the host object behind it through the granted channel with the token. |
| [`InvokeClient`](#invokeclient) | struct | Cap'n Proto client hook whose calls are `POST /invoke` requests. |
| [`CapDescriptor`](#capdescriptor) | enum | One entry of the `X-Bookclerk-Caps` table. Entry `i` describes capability index `i` of the message beside it. |
| [`method_headers`](#method_headers) | function | Header values for one `(interface, method)` pair. |
| [`MAX_INVOKE_BODY_BYTES`](#max_invoke_body_bytes) | constant | Largest `/invoke` request or reply body. |

## Structs

### GrantCap

```rust
pub struct GrantCap {
    /* private fields */
}
```

A capability that exists only to be described: it is never callable from
this process. Placed in a `$Params` capability table it is serialized as
its [`CapDescriptor`](#capdescriptor) in `X-Bookclerk-Caps`, and the isolate reaches the
host object behind it through the granted channel with the token.

#### GrantCap::new

```rust
pub fn new(descriptor: CapDescriptor) -> Self
```

Registers a new grant capability.

#### GrantCap::client

```rust
pub fn client<C: FromClientHook>(descriptor: CapDescriptor) -> C
```

Builds a typed Cap'n client (e.g. `source::Client`) over a new grant
capability.

#### GrantCap::descriptor

```rust
pub fn descriptor(&self) -> &CapDescriptor
```

The descriptor this capability serializes as.

Implements: `ClientHook`, `Clone`

### InvokeClient

```rust
pub struct InvokeClient {
    /* private fields */
}
```

Cap'n Proto client hook whose calls are `POST /invoke` requests.

#### InvokeClient::new

```rust
pub fn new(http: BridgeHttp, context: Option<String>) -> Self
```

A hook for calls carrying `context` (the bridge JSON of one `open`), or
none for `PluginWorker.*`.

#### InvokeClient::with_keepalive

```rust
pub fn with_keepalive(
    http: BridgeHttp,
    context: Option<String>,
    keepalive: Option<Rc<dyn Any>>,
) -> Self
```

Like [`new`](#invokeclientnew), also pinning `keepalive` for as long as any
client derived from this hook (including reply capabilities) lives.

#### InvokeClient::typed

```rust
pub fn typed<C: FromClientHook>(&self) -> C
```

Typed client (`content_source::Client`, `job_runner::Client`, …) over
this hook.

Implements: `ClientHook`, `Clone`

## Enums

### CapDescriptor

```rust
pub enum CapDescriptor {
    Source { token: String },
    Destination { token: String },
    Progress { token: String },
    Cancellation { token: String },
    EventPublisher { token: String },
    GuestDatabase { token: String, binding: String },
    AdapterSession { id: String },
}
```

One entry of the `X-Bookclerk-Caps` table. Entry `i` describes capability
index `i` of the message beside it.

**Variants**

- `Source`: `Source` → granted `GET /open`.
  - `token`: Grant token.
- `Destination`: `Destination` → granted `PUT /put`.
  - `token`: Grant token.
- `Progress`: `ProgressSink` → granted `POST /progress`.
  - `token`: Grant token.
- `Cancellation`: `Cancellation` → granted `GET /cancel` long poll.
  - `token`: Grant token.
- `EventPublisher`: `EventPublisher` → granted `POST /events/publish`.
  - `token`: Grant token.
- `GuestDatabase`: `GuestDatabase` → granted `POST /db/execute`.
  - `token`: Grant token.
  - `binding`: `[[databases]]` binding name.
- `AdapterSession`: Isolate-served `AdapterDatabaseSession`; later calls carry `id` in
  `X-Bookclerk-Target`.
  - `id`: Isolate-side object id.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### method_headers

```rust
pub fn method_headers(
    interface_id: u64,
    method_id: u16,
) -> Option<(&'static str, &'static str)>
```

Header values for one `(interface, method)` pair.

Returns `None` for interfaces that are not routed over `/invoke`
(`Source`, `ByteSource`, `EventPublisher`, `GuestDatabase`, …) and for
ordinals outside the schema.

## Constants

### MAX_INVOKE_BODY_BYTES

```rust
pub const MAX_INVOKE_BODY_BYTES: u32 = _;
```

Largest `/invoke` request or reply body.

Twice the scalar ceiling so an `EventConsumer.event` batch or a
`JobRunner.job` invocation with a checkpoint still fits; equal to the
isolate codec's traversal budget (`MAX_TRAVERSAL_WORDS` words in
`db-capnp.ts`), so anything larger would be rejected there anyway.
