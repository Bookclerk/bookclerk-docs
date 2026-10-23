**bookclerk_plugin_source_audible > auth**

# Module: auth

## Contents

**Structs**

- [`AuthLoginOptions`](#authloginoptions) - Inputs for interactive Audible OAuth (SPA Accounts connect / CLI login helpers).
- [`AuthSession`](#authsession) - Metadata returned after a successful Audible OAuth session.

**Enums**

- [`LoginMode`](#loginmode) - How to complete the browser sign-in.
- [`LoginProgress`](#loginprogress) - Progress events emitted during login (for CLI / TUI).

**Functions**

- [`begin_login`](#begin_login) - Runs Audible OAuth via audible-rs and returns [`AuthSession`](auth.md#bookclerk_plugin_source_audibleauthauthsession) metadata.

---

## bookclerk_plugin_source_audible::auth::AuthLoginOptions

*Struct*

Inputs for interactive Audible OAuth (SPA Accounts connect / CLI login helpers).

Prefer Accounts UI for operator flows; these options also drive guest
`loginStart` / in-process [`begin_login`](auth.md#bookclerk_plugin_source_audibleauthbegin_login).

**Fields:**
- `marketplace: String` - Audible marketplace / locale code (`us`, `uk`, `de`, …).
- `label: Option<String>` - Optional operator-facing account label stored with the secret.
- `callback_bind: std::net::SocketAddr` - Bind address for the local OAuth redirect receiver (`127.0.0.1:0` = ephemeral).
- `response_url: Option<String>` - Pre-captured redirect URL (external mode / paste); skips waiting on the server.
- `show_qr: bool` - When true, emit QR text alongside the authorize URL for headless terminals.
- `qr_mode: crate::qr::QrRenderMode` - How to render the QR payload ([`QrRenderMode`](qr.md#bookclerk_plugin_source_audibleqrqrrendermode)).
- `mode: LoginMode` - Whether to run a local callback server or require an external redirect paste.
- `timeout_secs: u64` - Seconds to wait for LoginServer capture before timing out.
- `audible_username: bool` - Pre-merger Audible username login (DE/US/UK only).
- `force: bool` - Overwrite an existing sealed authenticator for the same account id.
- `scope: Option<bookclerk_library::SourceScope>` - When `Some`, credentials are persisted via the Audible

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> AuthLoginOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::auth::AuthSession

*Struct*

Metadata returned after a successful Audible OAuth session.

Callers that pass `AuthLoginOptions::scope` already have credentials in
`encrypted_secrets`; guest callers export the authenticator separately.

**Fields:**
- `account_id: String` - Stable Bookclerk account id for library rows and secret lookup.
- `marketplace: String` - Marketplace used for the login (`us`, `uk`, …).
- `label: Option<String>` - Optional operator-facing label from `AuthLoginOptions::label`.
- `customer_id: Option<String>` - Audible customer id from the authenticator when present.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AuthSession`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::auth::LoginMode

*Enum*

How to complete the browser sign-in.

**Variants:**
- `Server` - Local reverse-proxy + QR (headless / SSH friendly). Default.
- `External` - Print authorize URL; read redirect from `--response-url` or stdin.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> LoginMode`
- **PartialEq**
  - `fn eq(self: &Self, other: &LoginMode) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> LoginMode`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::auth::LoginProgress

*Enum*

Progress events emitted during login (for CLI / TUI).

**Variants:**
- `LoginUrl{ url: String, qr: Option<String> }` - Browser URL the operator should open (optional pre-rendered QR text).
- `CallbackListening{ addr: std::net::SocketAddr }` - Local callback server is listening.
- `WaitingForCallback` - Waiting for the OAuth redirect / callback.
- `Completed{ account_id: String }` - Login finished for this account id.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> LoginProgress`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_audible::auth::begin_login

*Function*

Runs Audible OAuth via audible-rs and returns [`AuthSession`](auth.md#bookclerk_plugin_source_audibleauthauthsession) metadata.

When `AuthLoginOptions::scope` is `Some`, credentials are persisted to
`encrypted_secrets`. When `None` (guest / external plugin path), login
completes and returns session metadata only — the caller exports the
authenticator separately.

# Arguments

* `opts` - Marketplace, callback bind, login mode, and optional DB scope.
* `on_progress` - Callback for URL / QR / waiting / completed events (CLI/TUI).

# Errors

Returns `AudibleError::Auth` when locale, device registration, or OAuth fails.

```rust
fn begin_login<impl FnMut(LoginProgress)>(opts: AuthLoginOptions, on_progress: impl Trait) -> crate::error::Result<AuthSession>
```



