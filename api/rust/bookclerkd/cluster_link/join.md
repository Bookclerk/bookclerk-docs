**bookclerkd > cluster_link > join**

# Module: cluster_link::join

## Contents

**Structs**

- [`CallbackBody`](#callbackbody) - `POST /api/config/join/callback` body.
- [`InspectBody`](#inspectbody) - `POST /api/config/join/inspect` body.
- [`InviteBody`](#invitebody) - `POST /api/config/join/invite` body.
- [`LeaveBody`](#leavebody) - `POST /api/config/join/leave` body.
- [`SharingBody`](#sharingbody) - `POST /api/config/join/sharing` body.
- [`StartBody`](#startbody) - `POST /api/config/join/start` body.

**Functions**

- [`get_join_sharing`](#get_join_sharing) - `GET /api/config/join/sharing`: Settings → Library for the operator or
- [`get_join_status`](#get_join_status) - `GET /api/config/join`.
- [`post_join_callback`](#post_join_callback) - `POST /api/config/join/callback`: redeems the library's code with the
- [`post_join_inspect`](#post_join_inspect) - `POST /api/config/join/inspect`.
- [`post_join_invite`](#post_join_invite) - `POST /api/config/join/invite`: join with a pasted branch invite. The
- [`post_join_leave`](#post_join_leave) - `POST /api/config/join/leave`: Leave library. The leave runs on its own
- [`post_join_sharing`](#post_join_sharing) - `POST /api/config/join/sharing`: Settings → Library's two sharing
- [`post_join_start`](#post_join_start) - `POST /api/config/join/start`.

---

## bookclerkd::cluster_link::join::CallbackBody

*Struct*

`POST /api/config/join/callback` body.

**Fields:**
- `code: String` - The code from the library.
- `state: String` - The `state` it came back with.
- `error: Option<String>` - The library's `error` (`access_denied`), when it sent one.
- `connection: Option<String>` - The connection type the library's user chose (`clerk`; a branch

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::cluster_link::join::InspectBody

*Struct*

`POST /api/config/join/inspect` body.

**Fields:**
- `address: String` - The library's address (`https://owner.example`, `owner.lan:8443`).

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::cluster_link::join::InviteBody

*Struct*

`POST /api/config/join/invite` body.

**Fields:**
- `link: String` - The whole branch invite (`https://<library>/enroll#bcn1...`).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::cluster_link::join::LeaveBody

*Struct*

`POST /api/config/join/leave` body.

**Fields:**
- `cluster_id: String` - The library (owner cluster id) to leave.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::join::SharingBody

*Struct*

`POST /api/config/join/sharing` body.

**Fields:**
- `cluster_id: String` - The library (owner cluster id).
- `share_books: bool` - "Share this branch's books with *library*".
- `include_branches: bool` - "Include books from my own branches".

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::cluster_link::join::StartBody

*Struct*

`POST /api/config/join/start` body.

**Fields:**
- `address: String` - The library's address.
- `fingerprint: Option<String>` - The fingerprint the operator confirmed (self-signed libraries).
- `branch_name: Option<String>` - The name to join as (default: this machine's name).
- `clerk: bool` - Offer to join as a clerk (an owner node that shares the library's

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::join::get_join_sharing

*Function*

`GET /api/config/join/sharing`: Settings → Library for the operator or
an Owner: every library this install is a branch of, with its two
sharing settings (`{clusters: [...]}`).

```rust
fn get_join_sharing(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>) -> axum::response::Response
```



## bookclerkd::cluster_link::join::get_join_status

*Function*

`GET /api/config/join`.

```rust
fn get_join_status(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> axum::response::Response
```



## bookclerkd::cluster_link::join::post_join_callback

*Function*

`POST /api/config/join/callback`: redeems the library's code with the
sign-in's sealed verifier and keys. The pending sign-in is kept when the
redeem fails for a reason a retry can fix (answered as 502
`join_retry`; the page offers Try again with the same code), and ended
on success, refusal, or anything else.

```rust
fn post_join_callback(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<CallbackBody>) -> axum::response::Response
```



## bookclerkd::cluster_link::join::post_join_inspect

*Function*

`POST /api/config/join/inspect`.

```rust
fn post_join_inspect(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<InspectBody>) -> axum::response::Response
```



## bookclerkd::cluster_link::join::post_join_invite

*Function*

`POST /api/config/join/invite`: join with a pasted branch invite. The
link is redeemed once and dropped; it is never logged or echoed. A
library slower than `REDEEM_TIMEOUT` is answered 504 `join_pending`
while the join carries on and starts its client when it lands.

```rust
fn post_join_invite(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<InviteBody>) -> axum::response::Response
```



## bookclerkd::cluster_link::join::post_join_leave

*Function*

`POST /api/config/join/leave`: Leave library. The leave runs on its own
task, so a request cut short (the client gone, or the API deadline)
still records it once the library has been told.

```rust
fn post_join_leave(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<LeaveBody>) -> axum::response::Response
```



## bookclerkd::cluster_link::join::post_join_sharing

*Function*

`POST /api/config/join/sharing`: Settings → Library's two sharing
settings for one library this install is a branch of. The operator or
an Owner (verified from the stored role); anyone else gets 403.

```rust
fn post_join_sharing(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::Json<SharingBody>) -> axum::response::Response
```



## bookclerkd::cluster_link::join::post_join_start

*Function*

`POST /api/config/join/start`.

```rust
fn post_join_start(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::Json<StartBody>) -> axum::response::Response
```



