**bookclerkd > live**

# Module: live

## Contents

**Functions**

- [`get_live`](#get_live) - `GET /api/live`.
- [`origin_allowed`](#origin_allowed) - True when the request's `Origin` is exactly the origin this socket
- [`poke`](#poke) - Tells every open socket in this process that `topic` changed, without

**Constants**

- [`CLOSE_UNAUTHORIZED`](#close_unauthorized) - Close code: the session is no longer valid.
- [`LIVE_PATH`](#live_path) - Path of the socket.
- [`LIVE_SUBPROTOCOL`](#live_subprotocol) - Subprotocol a client may ask for (accepted, not required).
- [`POLL_INTERVAL`](#poll_interval) - How often revisions are read.
- [`REAUTH_INTERVAL`](#reauth_interval) - How often the caller's session is checked again.

---

## bookclerkd::live::CLOSE_UNAUTHORIZED

*Constant*: `u16`

Close code: the session is no longer valid.



## bookclerkd::live::LIVE_PATH

*Constant*: `&str`

Path of the socket.



## bookclerkd::live::LIVE_SUBPROTOCOL

*Constant*: `&str`

Subprotocol a client may ask for (accepted, not required).



## bookclerkd::live::POLL_INTERVAL

*Constant*: `std::time::Duration`

How often revisions are read.



## bookclerkd::live::REAUTH_INTERVAL

*Constant*: `std::time::Duration`

How often the caller's session is checked again.



## bookclerkd::live::get_live

*Function*

`GET /api/live`.

```rust
fn get_live(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, request: axum::extract::Request) -> axum::response::Response
```



## bookclerkd::live::origin_allowed

*Function*

True when the request's `Origin` is exactly the origin this socket
serves: `public_origin` when configured, else `http://<Host>` for a
loopback `Host` and no forwarding headers.

```rust
fn origin_allowed(headers: &axum::http::HeaderMap, public_origin: Option<&str>) -> bool
```



## bookclerkd::live::poke

*Function*

Tells every open socket in this process that `topic` changed, without
waiting for the next revision read.

```rust
fn poke(topic: bookclerk_library::control_plane::LiveTopic)
```



