# bookclerk_workerd::bridge_http

Minimal HTTP/1.1 client for the workerd bridge (JSON control plane, Cap'n
Proto `/invoke` bodies, and streamed object bodies).

| Item | Kind | Summary |
| --- | --- | --- |
| [`BridgeHttp`](#bridgehttp) | struct | Loopback HTTP client for the isolate bridge. |
| [`CapnpResponse`](#capnpresponse) | struct | Fully buffered reply of `BridgeHttp::capnp_post`. |
| [`generate_bridge_token`](#generate_bridge_token) | function | Generate a 32-byte hex bearer token for one isolate (`BRIDGE_TOKEN`). |
| [`check_health`](#check_health) | function | `GET /health` on the bridge at `127.0.0.1:<port>` with `token`, through `BridgeHttp::health`. |

## Structs

### BridgeHttp

```rust
pub struct BridgeHttp {
    pub port: u16,
    pub token: String,
}
```

Loopback HTTP client for the isolate bridge.

It writes HTTP/1.1 straight to a `TcpStream` on `127.0.0.1`, so proxy
environment variables (`ALL_PROXY`, `HTTPS_PROXY`, `HTTP_PROXY`) never
route a request, and the bearer token never leaves loopback.

**Fields**

- `port`: Bridge TCP port.
- `token`: Bearer token (`BRIDGE_TOKEN`).

#### BridgeHttp::health

```rust
pub async fn health(&self) -> Result<()>
```

`GET /health`: whether the bridge is up and accepts this token.

##### Errors

Returns transport failures, `bridge unauthorized` on 401, and any other
non-2xx status as `bridge HTTP <status>`.

#### BridgeHttp::json_post

```rust
pub async fn json_post(&self, path: &str, body: &Value) -> Result<Value>
```

POST JSON and parse a JSON object response.

##### Errors

Returns transport, HTTP, or JSON failures, including bridge `{error}`.

#### BridgeHttp::get_stream

```rust
pub async fn get_stream(
    &self,
    path: &str,
) -> Result<(ObjectMetadata, Pin<Box<dyn AsyncRead + Send>>)>
```

GET a streamed object body plus metadata headers.

##### Errors

Returns transport or HTTP failures.

#### BridgeHttp::get_stream_headers

```rust
pub async fn get_stream_headers(
    &self,
    path: &str,
    extra: &[(&str, &str)],
) -> Result<(ObjectMetadata, Pin<Box<dyn AsyncRead + Send>>)>
```

GET a streamed object body plus metadata headers, with extra request headers.

##### Errors

Returns transport or HTTP failures.

#### BridgeHttp::put_stream

```rust
pub async fn put_stream(
    &self,
    path: &str,
    body: Pin<Box<dyn AsyncRead + Send>>,
    content_type: Option<&str>,
    content_length: Option<u64>,
) -> Result<PutResult>
```

PUT a streamed body.

##### Errors

Returns transport, HTTP, or JSON failures.

#### BridgeHttp::put_stream_headers

```rust
pub async fn put_stream_headers(
    &self,
    path: &str,
    body: Pin<Box<dyn AsyncRead + Send>>,
    content_type: Option<&str>,
    content_length: Option<u64>,
    extra: &[(&str, &str)],
) -> Result<PutResult>
```

PUT a streamed body with extra request headers.

##### Errors

Returns transport, HTTP, or JSON failures.

#### BridgeHttp::capnp_post

```rust
pub async fn capnp_post(
    &self,
    path: &str,
    extra: &[(&str, &str)],
    body: &[u8],
    max_body_bytes: u32,
) -> Result<CapnpResponse>
```

POST an unpacked Cap'n Proto message and read the whole reply.

Unlike [`json_post`](#bridgehttpjson_post) this does not interpret the
status or the body: `/invoke` replies are Cap'n bytes on `200` and a
JSON error envelope otherwise, and the caller owns that mapping.

##### Errors

Returns transport failures, or `payload_too_large` when either body
exceeds `max_body_bytes`.

Implements: `Clone`

### CapnpResponse

```rust
pub struct CapnpResponse {
    pub status: u16,
    pub headers: Vec<(String, String)>,
    pub body: Vec<u8>,
}
```

Fully buffered reply of [`BridgeHttp::capnp_post`](#bridgehttpcapnp_post).

**Fields**

- `status`: HTTP status code.
- `headers`: Response headers as received (names keep the bridge's casing).
- `body`: Whole response body.

#### CapnpResponse::header

```rust
pub fn header(&self, name: &str) -> Option<String>
```

First non-empty header value with the given name (case-insensitive).

## Functions

### generate_bridge_token

```rust
pub fn generate_bridge_token() -> String
```

Generate a 32-byte hex bearer token for one isolate (`BRIDGE_TOKEN`).

### check_health

```rust
pub async fn check_health(port: u16, token: &str) -> Result<()>
```

`GET /health` on the bridge at `127.0.0.1:<port>` with `token`, through
[`BridgeHttp::health`](#bridgehttphealth).

The launcher polls this until the isolate is up. Like every other bridge
request it never consults proxy environment variables, so the session
token stays on loopback.

#### Arguments

* `port` - The bridge's loopback TCP port.
* `token` - The session's `BRIDGE_TOKEN`.

#### Errors

The errors of [`BridgeHttp::health`](#bridgehttphealth).
