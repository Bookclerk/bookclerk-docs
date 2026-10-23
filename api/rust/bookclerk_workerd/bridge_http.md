**bookclerk_workerd > bridge_http**

# Module: bridge_http

## Contents

**Structs**

- [`BridgeHttp`](#bridgehttp) - Loopback HTTP client for the isolate bridge.
- [`CapnpResponse`](#capnpresponse) - Fully buffered reply of `BridgeHttp::capnp_post`.

**Functions**

- [`generate_bridge_token`](#generate_bridge_token) - Generate a 32-byte hex bearer token for one isolate (`BRIDGE_TOKEN`).

---

## bookclerk_workerd::bridge_http::BridgeHttp

*Struct*

Loopback HTTP client for the isolate bridge.

**Fields:**
- `port: u16` - Bridge TCP port.
- `token: String` - Bearer token (`BRIDGE_TOKEN`).

**Methods:**

- `fn json_post(self: &Self, path: &str, body: &serde_json::Value) -> Result<serde_json::Value>` - POST JSON and parse a JSON object response.
- `fn get_stream(self: &Self, path: &str) -> Result<(ObjectMetadata, Pin<Box<dyn AsyncRead>>)>` - GET a streamed object body plus metadata headers.
- `fn get_stream_headers(self: &Self, path: &str, extra: &[(&str, &str)]) -> Result<(ObjectMetadata, Pin<Box<dyn AsyncRead>>)>` - GET a streamed object body plus metadata headers, with extra request headers.
- `fn put_stream(self: &Self, path: &str, body: Pin<Box<dyn AsyncRead>>, content_type: Option<&str>, content_length: Option<u64>) -> Result<PutResult>` - PUT a streamed body.
- `fn put_stream_headers(self: &Self, path: &str, body: Pin<Box<dyn AsyncRead>>, content_type: Option<&str>, content_length: Option<u64>, extra: &[(&str, &str)]) -> Result<PutResult>` - PUT a streamed body with extra request headers.
- `fn capnp_post(self: &Self, path: &str, extra: &[(&str, &str)], body: &[u8], max_body_bytes: u32) -> Result<CapnpResponse>` - POST an unpacked Cap'n Proto message and read the whole reply.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> BridgeHttp`



## bookclerk_workerd::bridge_http::CapnpResponse

*Struct*

Fully buffered reply of `BridgeHttp::capnp_post`.

**Fields:**
- `status: u16` - HTTP status code.
- `headers: Vec<(String, String)>` - Response headers as received (names keep the bridge's casing).
- `body: Vec<u8>` - Whole response body.

**Methods:**

- `fn header(self: &Self, name: &str) -> Option<String>` - First non-empty header value with the given name (case-insensitive).



## bookclerk_workerd::bridge_http::generate_bridge_token

*Function*

Generate a 32-byte hex bearer token for one isolate (`BRIDGE_TOKEN`).

```rust
fn generate_bridge_token() -> String
```



