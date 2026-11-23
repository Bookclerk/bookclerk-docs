# bookclerk_plugin_abi::origin

Sans-IO checks for the `ORIGIN` binding and the `ingress` entrypoint.

A relay `ingress` plugin hands the host the head of every public request it
carries ([`OriginRequest`](generated.md#originrequest)) and, when it starts, the public origin it serves
([`IngressStarted`](generated.md#ingressstarted)). Both come from a guest, so the host checks them here
before anything reaches the router or the database; the host checks the
response head it sends back the same way. Keeping the rules over plain
values lets the fuzz target and the unit tests drive the code the host runs.

| Item | Kind | Summary |
| --- | --- | --- |
| [`CheckedOriginRequest`](#checkedoriginrequest) | struct | A checked `OriginRequest`: what the host dispatches into the router. |
| [`IngressOrigin`](#ingressorigin) | struct | The public origin an ingress plugin serves: `https://` and a lowercase DNS host name, nothing else. |
| [`OriginResponseHead`](#originresponsehead) | struct | The head of the library's response to one origin request, as the isolate bridge frames it (`encode_origin_frame`). |
| [`OriginConnectBody`](#originconnectbody) | struct | The body of a `POST /origin/connect` on the isolate bridge: one `env.ORIGIN.connect` call. |
| [`check_origin_request`](#check_origin_request) | function | Checks the head of one public request a relay plugin carries and keeps what the host passes on. |
| [`check_origin_connect`](#check_origin_connect) | function | Checks the head of one WebSocket upgrade a relay plugin carries (`Origin.connect`) and keeps what the host passes on. |
| [`check_origin_refusal`](#check_origin_refusal) | function | Checks the library's answer to an upgrade it did not accept before it goes back to a relay plugin. |
| [`check_origin_response`](#check_origin_response) | function | Checks the head of a response the host sends back to a relay plugin. |
| [`parse_ingress_origin`](#parse_ingress_origin) | function | Parses the public origin an ingress plugin reports from `start`. |
| [`encode_origin_frame`](#encode_origin_frame) | function | Frames `head` for the isolate bridge: a 4-byte big-endian length, then the head as JSON. The body follows the frame on the same stream. |
| [`split_origin_frame`](#split_origin_frame) | function | Splits a framed head off the front of `buf`. |
| [`decode_origin_request_frame`](#decode_origin_request_frame) | function | Decodes and checks a framed request head from an isolate. |
| [`decode_origin_connect_body`](#decode_origin_connect_body) | function | Decodes and checks a `POST /origin/connect` body from an isolate. |
| [`decode_origin_response_frame`](#decode_origin_response_frame) | function | Decodes and checks a framed response head (the isolate side reads these; the host only writes them, so this is for tests and fuzzing). |
| [`MAX_ORIGIN_METHOD_BYTES`](#max_origin_method_bytes) | constant | Longest request method, in bytes. |
| [`ORIGIN_FRAME_PREFIX_BYTES`](#origin_frame_prefix_bytes) | constant | Bytes of the length prefix in front of a framed head. |
| [`MAX_ORIGIN_FRAME_HEAD_BYTES`](#max_origin_frame_head_bytes) | constant | Largest framed head on the isolate bridge's origin route: the JSON of a head at its limits, with room for escaping every header byte and the target. |
| [`MAX_ORIGIN_CONNECT_BODY_BYTES`](#max_origin_connect_body_bytes) | constant | Largest `POST /origin/connect` body the isolate bridge reads: a head at its framed limit and an attachment at its limit in base64. |

## Structs

### CheckedOriginRequest

```rust
pub struct CheckedOriginRequest {
    pub method: String,
    pub target: String,
    pub headers: Vec<OriginHeader>,
    pub client: Option<IpAddr>,
}
```

A checked [`OriginRequest`]: what the host dispatches into the router.

**Fields**

- `method`: Request method, an HTTP token, as sent.
- `target`: Path and query in origin form.
- `headers`: Header fields in order, without the ignored ones.
- `client`: The client's IP address, when the provider reported one.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### IngressOrigin

```rust
pub struct IngressOrigin(/* private */);
```

The public origin an ingress plugin serves: `https://` and a lowercase
DNS host name, nothing else.

#### IngressOrigin::as_str

```rust
pub fn as_str(&self) -> &str
```

The origin as text, `https://<host>`.

#### IngressOrigin::host

```rust
pub fn host(&self) -> &str
```

The host name, without the scheme.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Hash`, `PartialEq`

### OriginResponseHead

```rust
pub struct OriginResponseHead {
    pub status: u16,
    pub headers: Vec<OriginHeader>,
}
```

The head of the library's response to one origin request, as the
isolate bridge frames it ([`encode_origin_frame`](#encode_origin_frame)).

**Fields**

- `status`: HTTP status code, 100 to 599.
- `headers`: Response header fields in order.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OriginConnectBody

```rust
pub struct OriginConnectBody {
    pub request: OriginRequest,
    pub attachment: Vec<u8>,
}
```

The body of a `POST /origin/connect` on the isolate bridge: one
`env.ORIGIN.connect` call.

**Fields**

- `request`: The upgrade request's head.
- `attachment`: The socket's initial attachment, base64 on the wire.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### check_origin_request

```rust
pub fn check_origin_request(request: &OriginRequest) -> Result<CheckedOriginRequest>
```

Checks the head of one public request a relay plugin carries and keeps
what the host passes on.

#### Arguments

* `request` - The head as the guest sent it.

#### Returns

The method, target and client address as checked, and the headers
without `host` and forwarding fields.

#### Errors

[`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the method is not a token, the
target is not origin form or too long, a header breaks a rule, or the
client address is not an IP address.

### check_origin_connect

```rust
pub fn check_origin_connect(request: &OriginRequest) -> Result<CheckedOriginRequest>
```

Checks the head of one WebSocket upgrade a relay plugin carries
(`Origin.connect`) and keeps what the host passes on.

The guest sends the request without its connection fields (`upgrade`
and `connection` are hop-by-hop and refused as on `Origin.fetch`); the
host makes its own handshake to the library.

#### Arguments

* `request` - The head as the guest sent it.

#### Returns

The request as [`check_origin_request`](#check_origin_request) keeps it, without the
handshake fields the host writes itself.

#### Errors

[`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) for anything [`check_origin_request`](#check_origin_request)
refuses, and for a method other than `GET`.

### check_origin_refusal

```rust
pub fn check_origin_refusal(refusal: &OriginRefusal) -> Result<()>
```

Checks the library's answer to an upgrade it did not accept before it
goes back to a relay plugin.

#### Errors

[`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the status is outside 200 to 599,
a header breaks a rule, or the body exceeds
[`MAX_ORIGIN_REFUSAL_BODY_BYTES`](README.md#max_origin_refusal_body_bytes).

### check_origin_response

```rust
pub fn check_origin_response(status: u16, headers: &[OriginHeader]) -> Result<()>
```

Checks the head of a response the host sends back to a relay plugin.

#### Errors

[`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when the status is outside 100 to 599
or a header breaks a rule.

### parse_ingress_origin

```rust
pub fn parse_ingress_origin(started: &IngressStarted) -> Result<IngressOrigin>
```

Parses the public origin an ingress plugin reports from `start`.

#### Arguments

* `started` - The plugin's answer.

#### Returns

The origin, `https://` and a lowercase host name of at least two labels,
at most `maxIngressOriginBytes`; the top label starts with a letter, so
nothing a URL parser reads as an IP address passes.

#### Errors

[`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) for any other scheme, a port, path,
query, user info, trailing dot or slash, uppercase, or a host that is not
a DNS name.

### encode_origin_frame

```rust
pub fn encode_origin_frame<T: Serialize>(head: &T) -> Result<Vec<u8>>
```

Frames `head` for the isolate bridge: a 4-byte big-endian length, then
the head as JSON. The body follows the frame on the same stream.

#### Errors

`internal` when the head does not encode or is larger than
[`MAX_ORIGIN_FRAME_HEAD_BYTES`](#max_origin_frame_head_bytes).

### split_origin_frame

```rust
pub fn split_origin_frame(buf: &[u8]) -> Result<Option<(&[u8], &[u8])>>
```

Splits a framed head off the front of `buf`.

#### Returns

`Ok(None)` while `buf` holds less than the whole frame; otherwise the
head's JSON and the bytes after it (the start of the body).

#### Errors

`invalid_params` when the declared length exceeds
[`MAX_ORIGIN_FRAME_HEAD_BYTES`](#max_origin_frame_head_bytes), so a guest cannot make the host buffer
without bound.

### decode_origin_request_frame

```rust
pub fn decode_origin_request_frame(json: &[u8]) -> Result<CheckedOriginRequest>
```

Decodes and checks a framed request head from an isolate.

#### Errors

`invalid_params` when the JSON is not an [`OriginRequest`](generated.md#originrequest) or the head
fails [`check_origin_request`](#check_origin_request).

### decode_origin_connect_body

```rust
pub fn decode_origin_connect_body(json: &[u8]) -> Result<(CheckedOriginRequest, Vec<u8>)>
```

Decodes and checks a `POST /origin/connect` body from an isolate.

#### Returns

The head as [`check_origin_connect`](#check_origin_connect) keeps it, and the attachment.

#### Errors

`invalid_params` when the JSON is not an [`OriginConnectBody`](#originconnectbody), the head
fails [`check_origin_connect`](#check_origin_connect), or the attachment exceeds
`maxSocketAttachmentBytes`.

### decode_origin_response_frame

```rust
pub fn decode_origin_response_frame(json: &[u8]) -> Result<OriginResponseHead>
```

Decodes and checks a framed response head (the isolate side reads these;
the host only writes them, so this is for tests and fuzzing).

#### Errors

`invalid_params` when the JSON is not a head or fails
[`check_origin_response`](#check_origin_response).

## Constants

### MAX_ORIGIN_METHOD_BYTES

```rust
pub const MAX_ORIGIN_METHOD_BYTES: usize = 32;
```

Longest request method, in bytes.

### ORIGIN_FRAME_PREFIX_BYTES

```rust
pub const ORIGIN_FRAME_PREFIX_BYTES: usize = 4;
```

Bytes of the length prefix in front of a framed head.

### MAX_ORIGIN_FRAME_HEAD_BYTES

```rust
pub const MAX_ORIGIN_FRAME_HEAD_BYTES: usize = _;
```

Largest framed head on the isolate bridge's origin route: the JSON of a
head at its limits, with room for escaping every header byte and the
target.

### MAX_ORIGIN_CONNECT_BODY_BYTES

```rust
pub const MAX_ORIGIN_CONNECT_BODY_BYTES: usize = _;
```

Largest `POST /origin/connect` body the isolate bridge reads: a head at
its framed limit and an attachment at its limit in base64.
