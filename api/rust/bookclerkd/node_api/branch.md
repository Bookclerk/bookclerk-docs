**bookclerkd > node_api > branch**

# Module: node_api::branch

## Contents

**Structs**

- [`BranchAuthorizeParams`](#branchauthorizeparams) - The authorization request a consent is for (the query `/oidc/authorize`
- [`BranchConsentBody`](#branchconsentbody) - `POST /api/auth/branch-consent` body.
- [`PutBranches`](#putbranches) - `PUT /api/config/branches` body.
- [`RemoveBranchTitles`](#removebranchtitles) - `POST /api/config/branches/titles/remove` body.

**Enums**

- [`StepUp`](#stepup) - A fresh second factor for a clerk consent. `Debug` prints no code.

**Functions**

- [`get_branch_consent`](#get_branch_consent) - `GET /api/auth/branch-consent?<authorize params>`: what the consent page
- [`get_branches`](#get_branches) - `GET /api/config/branches`: this library's join settings: whether new
- [`get_info`](#get_info) - `GET /api/node/v1/info`.
- [`post_branch_consent`](#post_branch_consent) - `POST /api/auth/branch-consent`: approve or deny. Answers `{redirect}`,
- [`post_branch_consent_step_up`](#post_branch_consent_step_up) - `POST /api/auth/branch-consent/step-up`: starts a passkey step-up for a
- [`post_enroll_oauth`](#post_enroll_oauth) - `POST /api/node/v1/enroll/oauth`: redeem an approved branch sign-in.
- [`post_remove_branch_titles`](#post_remove_branch_titles) - `POST /api/config/branches/titles/remove`: removes the entries of
- [`put_branches`](#put_branches) - `PUT /api/config/branches`: turn "Let new branches join" on or off.

---

## bookclerkd::node_api::branch::BranchAuthorizeParams

*Struct*

The authorization request a consent is for (the query `/oidc/authorize`
forwarded to `/join/approve`).

**Fields:**
- `client_id: String` - Must be [`BRANCH_CLIENT_ID`](../../bookclerk_node_proto/join.md#bookclerk_node_protojoinbranch_client_id).
- `redirect_uri: String` - Where the code goes: loopback or https.
- `response_type: String` - Must be `code`.
- `scope: Option<String>` - Must be [`BRANCH_SCOPE`](../../bookclerk_node_proto/join.md#bookclerk_node_protojoinbranch_scope).
- `state: Option<String>` - Echoed on the redirect.
- `code_challenge: String` - PKCE S256 challenge.
- `code_challenge_method: Option<String>` - Must be `S256`.
- `branch_name: Option<String>` - The name the branch asks to join as.
- `clerk_offer: Option<String>` - `1` when the joining install offers to join as a clerk

**Methods:**

- `fn clerk_offered(self: &Self) -> bool` - True when the joining install offered to join as a clerk.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> BranchAuthorizeParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::node_api::branch::BranchConsentBody

*Struct*

`POST /api/auth/branch-consent` body.

**Fields:**
- `params: BranchAuthorizeParams` - The authorization request.
- `approve: bool` - True to let the branch join; false to deny.
- `connection: Option<String>` - `branch` (default) or `clerk`.
- `step_up: Option<StepUp>` - The fresh second factor a clerk needs.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::node_api::branch::PutBranches

*Struct*

`PUT /api/config/branches` body.

**Fields:**
- `enrollment: bool` - "Let new branches join" (`core.host_enrollment.enabled`).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::node_api::branch::RemoveBranchTitles

*Struct*

`POST /api/config/branches/titles/remove` body.

**Fields:**
- `node_id: Option<String>` - Only this branch's entries; every branch's when absent.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::node_api::branch::StepUp

*Enum*

A fresh second factor for a clerk consent. `Debug` prints no code.

**Variants:**
- `Totp{ code: String }` - A current authenticator code.
- `Passkey{ challenge_id: String, credential: serde_json::Value }` - A user-verified passkey assertion for a challenge from

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut std::fmt::Formatter) -> std::fmt::Result`



## bookclerkd::node_api::branch::get_branch_consent

*Function*

`GET /api/auth/branch-consent?<authorize params>`: what the consent page
shows. 400 `invalid_request` for a malformed request (the page then
shows the problem and never redirects).

```rust
fn get_branch_consent(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::extract::Query<BranchAuthorizeParams>) -> axum::response::Response
```



## bookclerkd::node_api::branch::get_branches

*Function*

`GET /api/config/branches`: this library's join settings: whether new
branches may join, the transport mode, and the SPKI fingerprint a
joining operator confirms in `self_signed` mode.

```rust
fn get_branches(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>) -> axum::response::Response
```



## bookclerkd::node_api::branch::get_info

*Function*

`GET /api/node/v1/info`.

```rust
fn get_info(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, request: axum::extract::Request) -> axum::response::Response
```



## bookclerkd::node_api::branch::post_branch_consent

*Function*

`POST /api/auth/branch-consent`: approve or deny. Answers `{redirect}`,
the URL to send the browser to: the branch's callback with a code (or
`error=access_denied`). Approving needs an active signed-in user, host
enrollment on, and a connection the transport policy allows.

```rust
fn post_branch_consent(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, request: axum::extract::Request) -> axum::response::Response
```



## bookclerkd::node_api::branch::post_branch_consent_step_up

*Function*

`POST /api/auth/branch-consent/step-up`: starts a passkey step-up for a
clerk consent (an Owner with a passkey). Answers the WebAuthn request
options with a `challenge_id` to send back in the consent's `step_up`.

```rust
fn post_branch_consent_step_up(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, headers: axum::http::HeaderMap) -> axum::response::Response
```



## bookclerkd::node_api::branch::post_enroll_oauth

*Function*

`POST /api/node/v1/enroll/oauth`: redeem an approved branch sign-in.
Answers `{cluster_id, node_id}`; 403 `transport_insecure` for a refused
connection, 403 `enrollment_refused` for a code that does not redeem
(unknown, used, expired, another client's, a wrong verifier or
redirect URI, enrollment off, or the user no longer active).

```rust
fn post_enroll_oauth(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, request: axum::extract::Request) -> axum::response::Response
```



## bookclerkd::node_api::branch::post_remove_branch_titles

*Function*

`POST /api/config/branches/titles/remove`: removes the entries of
titles from a branch that stopped sharing or left. Delivered files stay.

```rust
fn post_remove_branch_titles(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::Json<RemoveBranchTitles>) -> axum::response::Response
```



## bookclerkd::node_api::branch::put_branches

*Function*

`PUT /api/config/branches`: turn "Let new branches join" on or off.

```rust
fn put_branches(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Extension<crate::auth::UserCaller>, __arg2: axum::Json<PutBranches>) -> axum::response::Response
```



