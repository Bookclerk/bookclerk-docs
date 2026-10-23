# bookclerk_plugin_sdk::http

Native guest HTTP through the workerd socket proxy.

When [`crate::SOCKET_PROXY_ENV`](net.md#socket_proxy_env) is unset (crate tests, host-owned D1),
this module wraps [`reqwest::Client`](https://docs.rs/reqwest/latest/reqwest/async_impl/client/struct.Client.html) over ambient TCP. When the proxy
is set, every request is `CONNECT` + rustls over the Unix or Windows
named-pipe proxy — nested `NetPolicy::Deny` guests must not call
`socket(AF_INET)`. When [`crate::NESTED_NATIVE_JAIL_ENV`](net.md#nested_native_jail_env) is `1` without a
proxy, construction fails closed rather than falling back to ambient TCP.

Fetch domain grants do **not** imply TCP. Native manifests must declare
`capabilities.network.tcp` (or rely on a host overlay) for each
`host:port` this client dials.

| Item | Kind | Summary |
| --- | --- | --- |
| [`IntoUrlArg`](#intourlarg) | trait | Something `Client::get` / `Client::request` can target. |
| [`Error`](#error) | struct | Errors from `Client` / `RequestBuilder`. |
| [`CookieJar`](#cookiejar) | struct | Shared cookie jar (reqwest path + proxied `cookie_store`). |
| [`ClientBuilder`](#clientbuilder) | struct | Builder for `Client`. |
| [`Client`](#client) | struct | HTTP client: ambient reqwest, or CONNECT+rustls through the socket proxy. |
| [`RequestBuilder`](#requestbuilder) | struct | In-flight request. |
| [`Response`](#response) | struct | Origin response. |
| [`ByteStream`](#bytestream) | struct | `Stream` of body chunks from `Response::bytes_stream`. |
| [`connect_tls`](#connect_tls) | function | CONNECT then rustls to `address` (HTTPS origins). |
| [`HttpResult`](#httpresult) | type alias | Result alias for guest HTTP. |

## Re-exports

- [`http::header`](https://docs.rs/http/latest/http/header/index.html)
- [`http::HeaderMap`](https://docs.rs/http/latest/http/header/map/struct.HeaderMap.html)
- [`http::HeaderName`](https://docs.rs/http/latest/http/header/name/struct.HeaderName.html)
- [`http::HeaderValue`](https://docs.rs/http/latest/http/header/value/struct.HeaderValue.html)
- [`http::Method`](https://docs.rs/http/latest/http/method/struct.Method.html)
- [`http::StatusCode`](https://docs.rs/http/latest/http/status/struct.StatusCode.html)

## Modules

- [`redirect`](http/redirect.md): Redirect follow policy (reqwest-compatible names).

## Traits

### IntoUrlArg

```rust
pub trait IntoUrlArg {
    fn into_url_arg(self) -> String;
}
```

Something [`Client::get`](#clientget) / [`Client::request`](#clientrequest) can target.

#### IntoUrlArg::into_url_arg

```rust
fn into_url_arg(self) -> String
```

Owned URL string.

Implemented by: `&str`, `&String`, `&Url`, `String`, `Url`

## Structs

### Error

```rust
pub struct Error(/* private */);
```

Errors from [`Client`](#client) / [`RequestBuilder`](#requestbuilder).

#### Error::is_timeout

```rust
pub fn is_timeout(&self) -> bool
```

True when the message looks like a timeout (retry hint).

#### Error::is_connect

```rust
pub fn is_connect(&self) -> bool
```

True when the message looks like a connect/DNS failure (retry hint).

#### Error::is_request

```rust
pub fn is_request(&self) -> bool
```

True when the error is a transport-class failure worth retrying.

Implements: `Debug`, `Display`, `Error`, `From<Error>`, `From<ParseError>`, `From<SdkError>`

### CookieJar

```rust
pub struct CookieJar {
    /* private fields */
}
```

Shared cookie jar (reqwest path + proxied `cookie_store`).

#### CookieJar::new

```rust
pub fn new() -> Arc<Self>
```

Empty jar shared across clients in one guest process.

#### CookieJar::add_cookie_str

```rust
pub fn add_cookie_str(&self, cookie: &str, url: &Url)
```

Seeds a `Set-Cookie` line (Amazon login init cookies).

Implements: `Clone`, `Default`

### ClientBuilder

```rust
pub struct ClientBuilder {
    /* private fields */
}
```

Builder for [`Client`](#client).

#### ClientBuilder::new

```rust
pub fn new() -> Self
```

Empty builder (10-hop redirects, no cookies).

#### ClientBuilder::timeout

```rust
pub fn timeout(self, timeout: Duration) -> Self
```

Overall request timeout (connect + headers + body).

#### ClientBuilder::connect_timeout

```rust
pub fn connect_timeout(self, timeout: Duration) -> Self
```

CONNECT / TCP timeout (ambient reqwest path).

#### ClientBuilder::read_timeout

```rust
pub fn read_timeout(self, timeout: Duration) -> Self
```

Alias for [`Self::timeout`](#clientbuildertimeout) (audible-rs `read_timeout`).

#### ClientBuilder::user_agent

```rust
pub fn user_agent(self, value: impl Into<String>) -> Self
```

Default `User-Agent`.

#### ClientBuilder::cookie_store

```rust
pub fn cookie_store(self, enable: bool) -> Self
```

Enable reqwest / proxied cookie jars (separate stores; one process is
either proxied or ambient).

#### ClientBuilder::cookie_provider

```rust
pub fn cookie_provider(self, jar: Arc<CookieJar>) -> Self
```

Shared jar for Magento-style login + follow-up clients.

#### ClientBuilder::redirect

```rust
pub fn redirect(self, policy: Policy) -> Self
```

Redirect policy.

#### ClientBuilder::default_headers

```rust
pub fn default_headers(self, headers: HeaderMap) -> Self
```

Default headers merged into every request (request wins on conflict).

#### ClientBuilder::no_gzip

```rust
pub fn no_gzip(self) -> Self
```

Do not advertise gzip/br/deflate (CloudFront CENC URLs 403 on those).

#### ClientBuilder::no_brotli

```rust
pub fn no_brotli(self) -> Self
```

See [`Self::no_gzip`](#clientbuilderno_gzip).

#### ClientBuilder::no_deflate

```rust
pub fn no_deflate(self) -> Self
```

See [`Self::no_gzip`](#clientbuilderno_gzip).

#### ClientBuilder::build

```rust
pub fn build(self) -> HttpResult<Client>
```

Builds the client.

##### Errors

Returns when nested Deny is set without a socket proxy, when the
ambient reqwest client cannot be constructed, or when the proxied
rustls config fails.

Implements: `Clone`, `Default`

### Client

```rust
pub struct Client {
    /* private fields */
}
```

HTTP client: ambient reqwest, or CONNECT+rustls through the socket proxy.

#### Client::new

```rust
pub fn new() -> Self
```

Default client (ambient or proxied from env).

##### Panics

Panics when [`crate::SOCKET_PROXY_ENV`](net.md#socket_proxy_env) or nested Deny is in force and
the proxied client cannot be built. Does not fall back to ambient TCP.
Ambient reqwest construction failure without those env vars also panics
only if the last-resort `reqwest::Client::new` cannot be created.

#### Client::builder

```rust
pub fn builder() -> ClientBuilder
```

Builder.

#### Client::get

```rust
pub fn get(&self, url: impl IntoUrlArg) -> RequestBuilder
```

`GET`.

#### Client::post

```rust
pub fn post(&self, url: impl IntoUrlArg) -> RequestBuilder
```

`POST`.

#### Client::put

```rust
pub fn put(&self, url: impl IntoUrlArg) -> RequestBuilder
```

`PUT`.

#### Client::delete

```rust
pub fn delete(&self, url: impl IntoUrlArg) -> RequestBuilder
```

`DELETE`.

#### Client::request

```rust
pub fn request(&self, method: Method, url: impl IntoUrlArg) -> RequestBuilder
```

Arbitrary method.

Implements: `Clone`, `Debug`, `Default`

### RequestBuilder

```rust
pub struct RequestBuilder {
    /* private fields */
}
```

In-flight request.

#### RequestBuilder::header

```rust
pub fn header<K, V>(self, key: K, value: V) -> Self
where
    HeaderName: TryFrom<K>,
    HeaderValue: TryFrom<V>,
```

Extra header.

#### RequestBuilder::headers

```rust
pub fn headers(self, headers: HeaderMap) -> Self
```

Merge headers (existing keys on `self` win).

#### RequestBuilder::bearer_auth

```rust
pub fn bearer_auth(self, token: impl AsRef<str>) -> Self
```

`Authorization: Bearer`.

#### RequestBuilder::json

```rust
pub fn json<T: Serialize>(self, value: &T) -> Self
```

JSON body + `Content-Type`.

#### RequestBuilder::form

```rust
pub fn form<T: Serialize>(self, value: &T) -> Self
```

`application/x-www-form-urlencoded`.

#### RequestBuilder::body

```rust
pub fn body(self, body: impl Into<Vec<u8>>) -> Self
```

Raw body bytes.

#### RequestBuilder::query

```rust
pub fn query<T: Serialize>(self, query: &T) -> Self
```

Query string pairs (serde_urlencoded).

Values are decoded after `serde_urlencoded` so the URL encoder emits
them once (reqwest-compatible). Storing the encoded form would turn
`,` into `%252C` and Audible catalog search into HTTP 400.

#### RequestBuilder::timeout

```rust
pub fn timeout(self, timeout: Duration) -> Self
```

Per-request timeout override.

#### RequestBuilder::send

```rust
pub async fn send(self) -> HttpResult<Response>
```

Dispatch.

##### Errors

Returns when the URL is invalid, CONNECT/TLS fails, or the origin errors.

### Response

```rust
pub struct Response {
    /* private fields */
}
```

Origin response.

#### Response::status

```rust
pub fn status(&self) -> StatusCode
```

HTTP status.

#### Response::headers

```rust
pub fn headers(&self) -> &HeaderMap
```

Response headers (after cookie extraction on the proxied path).

#### Response::url

```rust
pub fn url(&self) -> &Url
```

Final URL after redirects.

#### Response::content_length

```rust
pub fn content_length(&self) -> Option<u64>
```

`Content-Length` when the origin sent a valid header.

#### Response::error_for_status

```rust
pub fn error_for_status(self) -> HttpResult<Self>
```

Fail on 4xx/5xx.

##### Errors

Returns when the status is a client or server error.

#### Response::bytes

```rust
pub async fn bytes(self) -> HttpResult<Bytes>
```

Entire body.

##### Errors

Returns on I/O or decode failure.

#### Response::text

```rust
pub async fn text(self) -> HttpResult<String>
```

UTF-8 body (lossy).

##### Errors

Returns on I/O failure.

#### Response::json

```rust
pub async fn json<T: DeserializeOwned>(self) -> HttpResult<T>
```

JSON body.

##### Errors

Returns on I/O or JSON decode failure.

#### Response::chunk

```rust
pub async fn chunk(&mut self) -> HttpResult<Option<Bytes>>
```

Next body chunk (download loops).

##### Errors

Returns on I/O failure.

#### Response::bytes_stream

```rust
pub fn bytes_stream(self) -> ByteStream
```

Streaming body (audible-rs / throttled downloads).

### ByteStream

```rust
pub struct ByteStream {
    /* private fields */
}
```

[`Stream`](https://docs.rs/futures_core/latest/futures_core/stream/trait.Stream.html) of body chunks from [`Response::bytes_stream`](#responsebytes_stream).

Implements: `Stream`

## Functions

### connect_tls

```rust
pub async fn connect_tls(address: SocketAddress) -> Result<TlsStream<ProxyStream>>
```

CONNECT then rustls to `address` (HTTPS origins).

#### Errors

Returns when the proxy is missing, CONNECT is denied, or TLS fails.

## Type aliases

### HttpResult

```rust
pub type HttpResult<T> = Result<T, Error>;
```

Result alias for guest HTTP.
