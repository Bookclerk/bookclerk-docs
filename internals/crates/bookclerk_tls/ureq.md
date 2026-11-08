# bookclerk_tls::ureq

The `ureq` crate with the hosts' TLS: everything `ureq` exports, except
that the request functions (`get`, `post`, …), `run` and `agent` use
[`http_agent`](README.md#http_agent).

Import it in place of the crate (`use bookclerk_tls::ureq;`); call sites
then read like plain `ureq`. Build any other agent from
`Agent::config_builder()` with [`ureq_tls_config`](README.md#ureq_tls_config), never from
`Agent::new_with_defaults()`, which has no provider.

| Item | Kind | Summary |
| --- | --- | --- |
| [`agent`](#agent) | function | The shared agent, in place of `ureq::agent()`, whose fresh default agent has no crypto provider. |
| [`run`](#run) | function | Sends a prebuilt `http::Request` on the shared agent, in place of `ureq::run`, which uses a fresh default agent. |
| [`get`](#get) | function | Starts a `GET` request. |
| [`post`](#post) | function | Starts a `POST` request. |
| [`put`](#put) | function | Starts a `PUT` request. |
| [`delete`](#delete) | function | Starts a `DELETE` request. |
| [`head`](#head) | function | Starts a `HEAD` request. |
| [`options`](#options) | function | Starts an `OPTIONS` request. |
| [`connect`](#connect) | function | Starts a `CONNECT` request. |
| [`patch`](#patch) | function | Starts a `PATCH` request. |
| [`trace`](#trace) | function | Starts a `TRACE` request. |

## Re-exports

- [`::ureq::*`](https://docs.rs/ureq/latest//ureq/index.html)

## Functions

### agent

```rust
pub fn agent() -> Agent
```

The shared agent, in place of `ureq::agent()`, whose fresh default
agent has no crypto provider.

#### Returns

A clone of [`super::http_agent`](README.md#http_agent).

### run

```rust
pub fn run(request: Request<impl AsSendBody>) -> Result<Response<Body>, Error>
```

Sends a prebuilt `http::Request` on the shared agent, in place of
`ureq::run`, which uses a fresh default agent.

#### Arguments

* `request` - The request to send.

#### Returns

The response.

#### Errors

Any [`Error`](https://docs.rs/ureq/latest/ureq/error/enum.Error.html) the transfer raises.

### get

```rust
pub fn get<T>(uri: T) -> RequestBuilder<WithoutBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `GET` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### post

```rust
pub fn post<T>(uri: T) -> RequestBuilder<WithBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `POST` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### put

```rust
pub fn put<T>(uri: T) -> RequestBuilder<WithBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `PUT` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### delete

```rust
pub fn delete<T>(uri: T) -> RequestBuilder<WithoutBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `DELETE` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### head

```rust
pub fn head<T>(uri: T) -> RequestBuilder<WithoutBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `HEAD` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### options

```rust
pub fn options<T>(uri: T) -> RequestBuilder<WithoutBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts an `OPTIONS` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### connect

```rust
pub fn connect<T>(uri: T) -> RequestBuilder<WithoutBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `CONNECT` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### patch

```rust
pub fn patch<T>(uri: T) -> RequestBuilder<WithBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `PATCH` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.

### trace

```rust
pub fn trace<T>(uri: T) -> RequestBuilder<WithoutBody>
where
    Uri: TryFrom<T>,
    <Uri as TryFrom<T>>::Error: Into<Error>,
```

Starts a `TRACE` request.

#### Arguments

* `uri` - The request target, anything that converts to a URI.

#### Returns

A request builder on the shared agent.
