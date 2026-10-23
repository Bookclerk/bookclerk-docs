**bookclerk_plugin_sdk > http**

# Module: http

## Contents

**Modules**

- [`redirect`](#redirect) - Redirect follow policy (reqwest-compatible names).

**Structs**

- [`ByteStream`](#bytestream) - [`Stream`](https://docs.rs/futures_core/latest/futures_core/stream/trait.Stream.html) of body chunks from `Response::bytes_stream`.
- [`Client`](#client) - HTTP client: ambient reqwest, or CONNECT+rustls through the socket proxy.
- [`ClientBuilder`](#clientbuilder) - Builder for [`Client`](http.md#bookclerk_plugin_sdkhttpclient).
- [`CookieJar`](#cookiejar) - Shared cookie jar (reqwest path + proxied `cookie_store`).
- [`Error`](#error) - Errors from [`Client`](http.md#bookclerk_plugin_sdkhttpclient) / [`RequestBuilder`](http.md#bookclerk_plugin_sdkhttprequestbuilder).
- [`RequestBuilder`](#requestbuilder) - In-flight request.
- [`Response`](#response) - Origin response.

**Functions**

- [`connect_tls`](#connect_tls) - CONNECT then rustls to `address` (HTTPS origins).

**Traits**

- [`IntoUrlArg`](#intourlarg) - Something `Client::get` / `Client::request` can target.

**Type Aliases**

- [`HttpResult`](#httpresult) - Result alias for guest HTTP.

---

## bookclerk_plugin_sdk::http::ByteStream

*Struct*

[`Stream`](https://docs.rs/futures_core/latest/futures_core/stream/trait.Stream.html) of body chunks from `Response::bytes_stream`.

**Trait Implementations:**

- **Stream**
  - `fn poll_next(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context) -> std::task::Poll<Option<<Self as >::Item>>`



## bookclerk_plugin_sdk::http::Client

*Struct*

HTTP client: ambient reqwest, or CONNECT+rustls through the socket proxy.

**Methods:**

- `fn new() -> Self` - Default client (ambient or proxied from env).
- `fn builder() -> ClientBuilder` - Builder.
- `fn get<impl IntoUrlArg>(self: &Self, url: impl Trait) -> RequestBuilder` - `GET`.
- `fn post<impl IntoUrlArg>(self: &Self, url: impl Trait) -> RequestBuilder` - `POST`.
- `fn put<impl IntoUrlArg>(self: &Self, url: impl Trait) -> RequestBuilder` - `PUT`.
- `fn delete<impl IntoUrlArg>(self: &Self, url: impl Trait) -> RequestBuilder` - `DELETE`.
- `fn request<impl IntoUrlArg>(self: &Self, method: Method, url: impl Trait) -> RequestBuilder` - Arbitrary method.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`
- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> Client`



## bookclerk_plugin_sdk::http::ClientBuilder

*Struct*

Builder for [`Client`](http.md#bookclerk_plugin_sdkhttpclient).

**Methods:**

- `fn new() -> Self` - Empty builder (10-hop redirects, no cookies).
- `fn timeout(self: Self, timeout: Duration) -> Self` - Overall request timeout (connect + headers + body).
- `fn connect_timeout(self: Self, timeout: Duration) -> Self` - CONNECT / TCP timeout (ambient reqwest path).
- `fn read_timeout(self: Self, timeout: Duration) -> Self` - Alias for `Self::timeout` (audible-rs `read_timeout`).
- `fn user_agent<impl Into<String>>(self: Self, value: impl Trait) -> Self` - Default `User-Agent`.
- `fn cookie_store(self: Self, enable: bool) -> Self` - Enable reqwest / proxied cookie jars (separate stores; one process is
- `fn cookie_provider(self: Self, jar: Arc<CookieJar>) -> Self` - Shared jar for Magento-style login + follow-up clients.
- `fn redirect(self: Self, policy: redirect::Policy) -> Self` - Redirect policy.
- `fn default_headers(self: Self, headers: HeaderMap) -> Self` - Default headers merged into every request (request wins on conflict).
- `fn no_gzip(self: Self) -> Self` - Do not advertise gzip/br/deflate (CloudFront CENC URLs 403 on those).
- `fn no_brotli(self: Self) -> Self` - See `Self::no_gzip`.
- `fn no_deflate(self: Self) -> Self` - See `Self::no_gzip`.
- `fn build(self: Self) -> HttpResult<Client>` - Builds the client.

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> ClientBuilder`



## bookclerk_plugin_sdk::http::CookieJar

*Struct*

Shared cookie jar (reqwest path + proxied `cookie_store`).

**Methods:**

- `fn new() -> Arc<Self>` - Empty jar shared across clients in one guest process.
- `fn add_cookie_str(self: &Self, cookie: &str, url: &Url)` - Seeds a `Set-Cookie` line (Amazon login init cookies).

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CookieJar`
- **Default**
  - `fn default() -> Self`



## bookclerk_plugin_sdk::http::Error

*Struct*

Errors from [`Client`](http.md#bookclerk_plugin_sdkhttpclient) / [`RequestBuilder`](http.md#bookclerk_plugin_sdkhttprequestbuilder).

**Tuple Struct**: `()`

**Methods:**

- `fn is_timeout(self: &Self) -> bool` - True when the message looks like a timeout (retry hint).
- `fn is_connect(self: &Self) -> bool` - True when the message looks like a connect/DNS failure (retry hint).
- `fn is_request(self: &Self) -> bool` - True when the error is a transport-class failure worth retrying.

**Traits:** Error

**Trait Implementations:**

- **From**
  - `fn from(err: url::ParseError) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Display**
  - `fn fmt(self: &Self, __formatter: & mut ::core::fmt::Formatter) -> ::core::fmt::Result`
- **From**
  - `fn from(err: reqwest::Error) -> Self`
- **From**
  - `fn from(err: SdkError) -> Self`



## bookclerk_plugin_sdk::http::HttpResult

*Type Alias*: `std::result::Result<T, Error>`

Result alias for guest HTTP.



## bookclerk_plugin_sdk::http::IntoUrlArg

*Trait*

Something `Client::get` / `Client::request` can target.

**Methods:**

- `into_url_arg`: Owned URL string.



## bookclerk_plugin_sdk::http::RequestBuilder

*Struct*

In-flight request.

**Methods:**

- `fn header<K, V>(self: Self, key: K, value: V) -> Self` - Extra header.
- `fn headers(self: Self, headers: HeaderMap) -> Self` - Merge headers (existing keys on `self` win).
- `fn bearer_auth<impl AsRef<str>>(self: Self, token: impl Trait) -> Self` - `Authorization: Bearer`.
- `fn json<T>(self: Self, value: &T) -> Self` - JSON body + `Content-Type`.
- `fn form<T>(self: Self, value: &T) -> Self` - `application/x-www-form-urlencoded`.
- `fn body<impl Into<Vec<u8>>>(self: Self, body: impl Trait) -> Self` - Raw body bytes.
- `fn query<T>(self: Self, query: &T) -> Self` - Query string pairs (serde_urlencoded).
- `fn timeout(self: Self, timeout: Duration) -> Self` - Per-request timeout override.
- `fn send(self: Self) -> HttpResult<Response>` - Dispatch.



## bookclerk_plugin_sdk::http::Response

*Struct*

Origin response.

**Methods:**

- `fn status(self: &Self) -> StatusCode` - HTTP status.
- `fn headers(self: &Self) -> &HeaderMap` - Response headers (after cookie extraction on the proxied path).
- `fn url(self: &Self) -> &Url` - Final URL after redirects.
- `fn content_length(self: &Self) -> Option<u64>` - `Content-Length` when the origin sent a valid header.
- `fn error_for_status(self: Self) -> HttpResult<Self>` - Fail on 4xx/5xx.
- `fn bytes(self: Self) -> HttpResult<Bytes>` - Entire body.
- `fn text(self: Self) -> HttpResult<String>` - UTF-8 body (lossy).
- `fn json<T>(self: Self) -> HttpResult<T>` - JSON body.
- `fn chunk(self: & mut Self) -> HttpResult<Option<Bytes>>` - Next body chunk (download loops).
- `fn bytes_stream(self: Self) -> ByteStream` - Streaming body (audible-rs / throttled downloads).



## bookclerk_plugin_sdk::http::connect_tls

*Function*

CONNECT then rustls to `address` (HTTPS origins).

# Errors

Returns when the proxy is missing, CONNECT is denied, or TLS fails.

```rust
fn connect_tls(address: crate::net::SocketAddress) -> crate::error::Result<tokio_rustls::client::TlsStream<crate::net::ProxyStream>>
```



## Module: redirect

Redirect follow policy (reqwest-compatible names).



