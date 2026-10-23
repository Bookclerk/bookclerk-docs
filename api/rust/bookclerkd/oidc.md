**bookclerkd > oidc**

# Module: oidc

## Contents

**Structs**

- [`AuthorizeQuery`](#authorizequery) - Query parameters for `GET /oidc/authorize` (authorization-code + PKCE).
- [`ConsentBody`](#consentbody) - Form body for `POST /oidc/authorize` after the user grants or denies consent.
- [`RevokeForm`](#revokeform) - Form body for `POST /oidc/revoke` (RFC 7009-style refresh-token revoke).
- [`TokenForm`](#tokenform) - `application/x-www-form-urlencoded` body for `POST /oidc/token`.

**Functions**

- [`redirect_uris_from_plugin_base`](#redirect_uris_from_plugin_base) - Build `{origin}{callback_path}` from an operator-set plugin server URL.
- [`router`](#router) - Mounts discovery, authorize, token, userinfo, revoke, and client admin APIs.
- [`sync_plugin_oidc_clients`](#sync_plugin_oidc_clients) - Materialize plugin-owned OIDC clients from guest RPC and `plugin.toml`.

---

## bookclerkd::oidc::AuthorizeQuery

*Struct*

Query parameters for `GET /oidc/authorize` (authorization-code + PKCE).

**Fields:**
- `client_id: String` - Registered OIDC client id (must exist in the library store).
- `redirect_uri: String` - Redirect URI; must match a URI registered for the client.
- `response_type: String` - OAuth `response_type`; only `code` is accepted.
- `scope: Option<String>` - Space-separated scopes; defaults to `openid profile` when omitted.
- `state: Option<String>` - Opaque CSRF value echoed back on the redirect.
- `code_challenge: String` - PKCE S256 challenge (base64url SHA-256 of the verifier).
- `code_challenge_method: Option<String>` - PKCE method; omitted is treated as `plain` and then rejected.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::oidc::ConsentBody

*Struct*

Form body for `POST /oidc/authorize` after the user grants or denies consent.

**Fields:**
- `client_id: String` - Registered OIDC client id (must exist in the library store).
- `redirect_uri: String` - Redirect URI; must match a URI registered for the client.
- `response_type: String` - OAuth `response_type`; only `code` is accepted.
- `scope: Option<String>` - Space-separated scopes; defaults to `openid profile` when omitted.
- `state: Option<String>` - Opaque CSRF value echoed back on the redirect.
- `code_challenge: String` - PKCE S256 challenge (base64url SHA-256 of the verifier).
- `code_challenge_method: Option<String>` - PKCE method; omitted is treated as `plain` and then rejected.
- `consent: Option<String>` - `deny` aborts with `access_denied`; any other value proceeds to issue a code.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::oidc::RevokeForm

*Struct*

Form body for `POST /oidc/revoke` (RFC 7009-style refresh-token revoke).

**Fields:**
- `token: String` - Token to revoke; treated as a refresh token (hashed before lookup).
- `token_type_hint: Option<String>` - Optional hint ignored; this server only stores refresh tokens.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::oidc::TokenForm

*Struct*

`application/x-www-form-urlencoded` body for `POST /oidc/token`.

**Fields:**
- `grant_type: String` - Grant: `authorization_code` or `refresh_token`; anything else is 400.
- `code: Option<String>` - Authorization code from the redirect (`authorization_code` grant).
- `redirect_uri: Option<String>` - Must match the URI bound to the consumed authorization code.
- `client_id: Option<String>` - Client id; required for code exchange, optional (but checked) on refresh.
- `client_secret: Option<String>` - Client secret for confidential clients (`client_secret_post`).
- `code_verifier: Option<String>` - PKCE verifier; SHA-256 must match the stored challenge.
- `refresh_token: Option<String>` - Refresh token to rotate (`refresh_token` grant).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::oidc::redirect_uris_from_plugin_base

*Function*

Build `{origin}{callback_path}` from an operator-set plugin server URL.

Loopback hosts also register the usual `localhost` / `127.0.0.1` alias.

```rust
fn redirect_uris_from_plugin_base(base_url: &str, callback_path: &str) -> Vec<String>
```



## bookclerkd::oidc::router

*Function*

Mounts discovery, authorize, token, userinfo, revoke, and client admin APIs.

```rust
fn router(state: std::sync::Arc<crate::api::AppState>) -> axum::Router
```



## bookclerkd::oidc::sync_plugin_oidc_clients

*Function*

Materialize plugin-owned OIDC clients from guest RPC and `plugin.toml`.

Used at startup and after config reload. New plugin clients start disabled;
existing rows keep their enable flag while redirects refresh from plugin
settings.

```rust
fn sync_plugin_oidc_clients(state: &crate::api::AppState) -> bookclerk_library::Result<()>
```



