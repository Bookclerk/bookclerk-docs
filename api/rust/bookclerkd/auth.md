**bookclerkd > auth**

# Module: auth

## Contents

**Structs**

- [`AuthMeImpersonating`](#authmeimpersonating) - Target user shown while an operator session is impersonating.
- [`AuthMeResponse`](#authmeresponse) - JSON body for `GET /api/auth/me`.
- [`AuthMeUser`](#authmeuser) - First-party user snapshot attached to `/me` when a `users` row is linked.
- [`AuthSecondFactor`](#authsecondfactor) - Host MFA policy plus whether this user has TOTP and/or passkeys.
- [`AuthSsoPicture`](#authssopicture) - HTTPS picture from a linked identity provider.
- [`BootstrapRequest`](#bootstraprequest) - Operator-only body for creating the first Owner when none exist.
- [`CreateUserRequest`](#createuserrequest) - Provisioner body for creating a user and optional invite ticket.
- [`ElevateRequest`](#elevaterequest) - JSON body for Owner password re-auth before issuing an elevated operator cookie.
- [`ImpersonateRequest`](#impersonaterequest) - JSON body naming the first-party user to impersonate.
- [`LoginRequest`](#loginrequest) - JSON body for `POST /api/auth/login` (operator token).
- [`OperatorAuthState`](#operatorauthstate) - In-memory operator token, session TTL, and login throttle for the daemon.
- [`PasswordLoginRequest`](#passwordloginrequest) - JSON body for local password login (login name or email plus password).
- [`PatchUserRequest`](#patchuserrequest) - Partial profile, role, or status patch for a first-party user.
- [`PortalMeInfo`](#portalmeinfo) - Portal identity fields exposed on `/me` for SSO / local-portal callers.
- [`SetPasswordRequest`](#setpasswordrequest) - JSON body for setting a password (self or provisioner target).

**Enums**

- [`NodeAdminCaller`](#nodeadmincaller) - Who is calling a node-administration route.
- [`UserCaller`](#usercaller) - Who is calling a route that acts for a user (host enrollment links).
- [`UserCallerRefusal`](#usercallerrefusal) - Why [`resolve_user_caller`](auth.md#bookclerkdauthresolve_user_caller) admitted nobody.

**Functions**

- [`bootstrap`](#bootstrap) - Operator-only bootstrap of the first Owner when none exist.
- [`caller_portal_identity`](#caller_portal_identity) - Resolve portal identity when the caller is not a pure operator.
- [`create_user`](#create_user) - Owner/admin or operator creates a user and optional invite magic link.
- [`create_user_claim_ticket`](#create_user_claim_ticket) - Mint a fresh local claim ticket for an active first-party user.
- [`delete_user`](#delete_user) - Delete a user and their personal data (wishlist / links / sessions); keep acquired books.
- [`elevate`](#elevate) - Owner portal session + password re-auth → short-lived elevated operator cookie.
- [`elevate_end`](#elevate_end) - End elevation by revoking the elevated operator session cookie.
- [`impersonate`](#impersonate) - Operator (or elevated) session starts impersonating a User.
- [`impersonate_end`](#impersonate_end) - Clear impersonation on the current operator session.
- [`list_sessions`](#list_sessions) - List sessions for the current principal only (not every operator session).
- [`list_users`](#list_users) - List first-party users (operator or administrator provisioner).
- [`login`](#login) - Validates the operator token (throttled, constant-time) and issues a hashed session cookie.
- [`logout`](#logout) - Revokes hashed operator and portal sessions and clears both cookies.
- [`me`](#me) - Resolves the caller and returns `/me` JSON; unauthenticated callers get 401.
- [`normalize_default_view`](#normalize_default_view) - Maps a prefs string onto a known SPA view; unknown values become `discover`.
- [`password_login`](#password_login) - Local password login → portal session cookie, or a TOTP challenge when enabled.
- [`patch_user`](#patch_user) - Patch role/status/display/login for a first-party user (provisioner).
- [`prefs_subject_for_caller`](#prefs_subject_for_caller) - Subject key + optional portal identity id for the caller's preferences row.
- [`require_operator_auth`](#require_operator_auth) - Middleware that admits operator sessions or Bearer tokens; impersonation is forbidden except ending it.
- [`require_operator_or_administrator_auth`](#require_operator_or_administrator_auth) - Operator token/session **or** an active Administrator user portal session.
- [`require_operator_or_node_owner_auth`](#require_operator_or_node_owner_auth) - Operator token/session **or** a portal user who may administer the nodes
- [`require_operator_or_owner_auth`](#require_operator_or_owner_auth) - Operator token/session **or** an Owner portal session.
- [`require_operator_or_portal_auth`](#require_operator_or_portal_auth) - Allow operator sessions **or** portal sessions (Discover / read-only library).
- [`require_operator_or_user_auth`](#require_operator_or_user_auth) - Operator token/session **or** any signed-in portal user with a user
- [`reset_user_password`](#reset_user_password) - Invalidate a user's password, revoke sessions, and mint a claim ticket for reset.
- [`resolve_user_caller`](#resolve_user_caller) - The [`UserCaller`](auth.md#bookclerkdauthusercaller) `headers` authenticate for `path`, exactly as
- [`revoke_session`](#revoke_session) - Revoke a session by id. Operators may revoke any operator session;
- [`set_password`](#set_password) - Set password for self (portal) or target user (operator/admin); revokes sessions.
- [`signin_methods`](#signin_methods) - Public sign-in picker: operator-token availability, SSO IdPs, integration logins.
- [`tray_handoff`](#tray_handoff) - Consume the pending tray ticket and set a localhost operator session cookie.
- [`tray_handoff_prepare`](#tray_handoff_prepare) - Mint a single-use tray Open Bookclerk ticket (Bearer, direct loopback only).

**Constants**

- [`ELEVATION_TTL`](#elevation_ttl) - Elevated Administrator→Operator session lifetime.
- [`OPERATOR_TOKEN_GRACE`](#operator_token_grace) - How long a previous operator token remains valid after rotate/reload.
- [`PORTAL_SESSION_COOKIE`](#portal_session_cookie) - HttpOnly cookie name for the portal session id (hashed before persist).
- [`SESSION_COOKIE`](#session_cookie) - HttpOnly cookie name for the operator session id (hashed before persist).

---

## bookclerkd::auth::AuthMeImpersonating

*Struct*

Target user shown while an operator session is impersonating.

**Fields:**
- `user_id: i64` - First-party `users.id` being impersonated.
- `display_name: Option<String>` - Display name of the impersonated user when the row exists.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerkd::auth::AuthMeResponse

*Struct*

JSON body for `GET /api/auth/me`.

**Fields:**
- `authenticated: bool` - Whether the request resolved to an operator, portal, or impersonated session.
- `role: Option<String>` - `operator`, `administrator`, or `member` (legacy clients may still send `portal`).
- `default_view: String` - Configured post-auth landing view (`discover` / `library` / `accounts`).
- `can_acquire: bool` - Whether this session may acquire / scan / manage jobs.
- `elevated: bool` - True when this operator session was created via Owner elevate.
- `impersonating: Option<AuthMeImpersonating>` - Present when the operator is impersonating a User.
- `portal: Option<PortalMeInfo>` - Linked portal identity when the caller is a portal (or impersonated) user.
- `user: Option<AuthMeUser>` - First-party user when the session is linked to a `users` row.
- `second_factor: Option<AuthSecondFactor>` - Password-login second-factor policy and this user's enrollment.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::AuthMeUser

*Struct*

First-party user snapshot attached to `/me` when a `users` row is linked.

**Fields:**
- `id: i64` - First-party `users.id`.
- `role: String` - Role string (`owner`, `administrator`, `member`).
- `display_name: Option<String>` - Optional human-readable name from the user row.
- `email: Option<String>` - Optional email; omitted from JSON when unset.
- `has_password: bool` - True when a local password hash is stored (invite users may be false).
- `has_avatar: bool` - True when a JPEG avatar is stored under `files_dir/avatars/{id}.*`.
- `avatar_source: String` - Selected picture source (`auto`, `monogram`, `gravatar`, `upload`, `sso:{id}`).
- `gravatar_hash: Option<String>` - SHA-256 hex of the contact email for Gravatar, when an email is set.
- `sso_pictures: Vec<AuthSsoPicture>` - IdP-supplied pictures the user can pick.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::AuthSecondFactor

*Struct*

Host MFA policy plus whether this user has TOTP and/or passkeys.

**Fields:**
- `required: bool` - `daemon.auth.require_second_factor`.
- `totp: bool` - Confirmed authenticator-app TOTP.
- `passkey_count: u64` - Number of registered passkeys.
- `enrolled: bool` - True when TOTP is enabled or at least one passkey exists.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerkd::auth::AuthSsoPicture

*Struct*

HTTPS picture from a linked identity provider.

**Fields:**
- `identity_id: i64` - `portal_identities.id`.
- `provider: String` - Identity-broker id (`oidc:google`, …).
- `picture_url: String` - HTTPS URL to display.
- `last_used_at: Option<String>` - RFC 3339 last portal-session activity for this identity, when any.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::BootstrapRequest

*Struct*

Operator-only body for creating the first Owner when none exist.

**Fields:**
- `display_name: Option<String>` - Optional display name; falls back to login or email.
- `login_name: Option<String>` - Optional local login name.
- `email: Option<String>` - Optional email.
- `password: Option<String>` - Optional password; hashed before persist and never stored in plaintext.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::CreateUserRequest

*Struct*

Provisioner body for creating a user and optional invite ticket.

**Fields:**
- `role: Option<String>` - Role to assign when permitted (`member` when omitted).
- `display_name: Option<String>` - Optional display name for the new user.
- `login_name: Option<String>` - Optional local login name for the new user.
- `email: Option<String>` - Optional email for the new user.
- `password: Option<String>` - Optional password; hashed before persist. Empty or omitted leaves the user passwordless.
- `mint_invite: bool` - When true (default), also mint an invite/claim ticket.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::ELEVATION_TTL

*Constant*: `std::time::Duration`

Elevated Administrator→Operator session lifetime.



## bookclerkd::auth::ElevateRequest

*Struct*

JSON body for Owner password re-auth before issuing an elevated operator cookie.

**Fields:**
- `password: String` - Owner account password (re-authentication).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::ImpersonateRequest

*Struct*

JSON body naming the first-party user to impersonate.

**Fields:**
- `user_id: i64` - Target `users.id` to impersonate.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::LoginRequest

*Struct*

JSON body for `POST /api/auth/login` (operator token).

**Fields:**
- `token: String` - Operator token from the client; compared in constant time and never persisted.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::NodeAdminCaller

*Enum*

Who is calling a node-administration route.

**Variants:**
- `Operator` - Operator token or session, or auth disabled: every node.
- `NodeOwner{ user_id: i64 }` - A portal user who owns nodes, or whose role (Host, Administrator,

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NodeAdminCaller`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeAdminCaller) -> bool`



## bookclerkd::auth::OPERATOR_TOKEN_GRACE

*Constant*: `std::time::Duration`

How long a previous operator token remains valid after rotate/reload.



## bookclerkd::auth::OperatorAuthState

*Struct*

In-memory operator token, session TTL, and login throttle for the daemon.

**Fields:**
- `token: String` - Current operator token compared in constant time; mismatch fails closed.
- `previous_token: Option<(String, std::time::Instant)>` - Prior token accepted until this deadline (rotate/reload overlap).
- `session_ttl: std::time::Duration` - Lifetime of a newly issued operator session cookie (at least one hour).
- `enabled: bool` - When false, operator auth is skipped and handlers treat the caller as operator.
- `login_max_failures: u32` - Failures allowed in one window before lockout (at least 1).
- `login_window: std::time::Duration` - Sliding window over which failures accumulate (defaults to the lockout duration).
- `login_lockout: std::time::Duration` - How long a client stays locked after exceeding `login_max_failures`.
- `login_attempts: tokio::sync::Mutex<std::collections::HashMap<String, LoginThrottleBucket>>` - Per-client throttle buckets keyed by resolved client IP.

**Methods:**

- `fn new(token: String, session_ttl_hours: u64, enabled: bool, login_max_failures: u32, login_lockout_secs: u64) -> Self` - Builds throttle and session settings from config, clamping TTL and lockout to safe minima.
- `fn install_token_grace_from(self: & mut Self, previous: &Self, grace: Duration)` - Accept `previous`'s token (and any still-valid grace token) for [`OPERATOR_TOKEN_GRACE`](auth.md#bookclerkdauthoperator_token_grace).
- `fn take_session_state_from(self: & mut Self, previous: &Self)` - Preserve in-memory login throttle buckets across auth reload swaps.
- `fn token_matches(self: &Self, candidate: &str) -> bool` - Constant-time compare against the current token or a still-valid grace token.
- `fn login_throttle_check(self: &Self, client_key: &str) -> Option<Duration>` - `None` = allowed; `Some(retry_after)` = locked out.
- `fn record_login_failure(self: &Self, client_key: &str) -> Option<Duration>` - Increments the client's failure count and returns lockout remaining when the cap is hit.
- `fn clear_login_failures(self: &Self, client_key: &str)` - Drops the client's throttle bucket after a successful login.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::PORTAL_SESSION_COOKIE

*Constant*: `&str`

HttpOnly cookie name for the portal session id (hashed before persist).



## bookclerkd::auth::PasswordLoginRequest

*Struct*

JSON body for local password login (login name or email plus password).

**Fields:**
- `login: String` - Login name or email.
- `password: String` - Password verified against the stored hash; mismatch fails closed and counts toward lockout.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::PatchUserRequest

*Struct*

Partial profile, role, or status patch for a first-party user.

**Fields:**
- `role: Option<String>` - Replacement role (`owner` / `administrator` / `member`) when present.
- `status: Option<String>` - Replacement status (`active` / `disabled`) when present.
- `display_name: Option<String>` - Replacement display name when present.
- `login_name: Option<String>` - Replacement local login name when present.
- `email: Option<String>` - Replacement email when present.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::PortalMeInfo

*Struct*

Portal identity fields exposed on `/me` for SSO / local-portal callers.

**Fields:**
- `identity_id: i64` - `portal_identities.id` for this session.
- `provider: String` - Identity-broker id (`local`, OIDC provider name, …).
- `external_user_id: String` - Provider-stable subject / external user id.
- `label: Option<String>` - Optional operator-facing label for the identity.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::SESSION_COOKIE

*Constant*: `&str`

HttpOnly cookie name for the operator session id (hashed before persist).



## bookclerkd::auth::SetPasswordRequest

*Struct*

JSON body for setting a password (self or provisioner target).

**Fields:**
- `password: String` - New password; hashed before persist. Empty is rejected.
- `current_password: Option<String>` - Required when the caller already has a password (not first-time invite setup).
- `user_id: Option<i64>` - Target `users.id` when a provisioner sets another user's password.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::auth::UserCaller

*Enum*

Who is calling a route that acts for a user (host enrollment links).

**Variants:**
- `Operator` - Operator token or session, or auth disabled.
- `User{ user_id: i64 }` - A signed-in portal user (or the user an operator impersonates). The

**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UserCaller`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &UserCaller) -> bool`



## bookclerkd::auth::UserCallerRefusal

*Enum*

Why [`resolve_user_caller`](auth.md#bookclerkdauthresolve_user_caller) admitted nobody.

**Variants:**
- `Status(axum::http::StatusCode)` - Answer with this status (401 signed out, 403 no user account).
- `Response(axum::response::Response)` - Answer with this response (a pending second-factor enrollment).



## bookclerkd::auth::bootstrap

*Function*

Operator-only bootstrap of the first Owner when none exist.

```rust
fn bootstrap(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::Json<BootstrapRequest>) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::caller_portal_identity

*Function*

Resolve portal identity when the caller is not a pure operator.

When an operator session is impersonating a User, returns that user's portal
identity so **prefs / Discover personalization** follow the target. Library
browsing is intentionally shared (not filtered by this identity).

```rust
fn caller_portal_identity(state: &crate::api::AppState, headers: &axum::http::HeaderMap) -> Option<bookclerk_library::PortalIdentity>
```



## bookclerkd::auth::create_user

*Function*

Owner/admin or operator creates a user and optional invite magic link.

```rust
fn create_user(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::Json<CreateUserRequest>) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::create_user_claim_ticket

*Function*

Mint a fresh local claim ticket for an active first-party user.

```rust
fn create_user_claim_ticket(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::extract::Path<i64>) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::delete_user

*Function*

Delete a user and their personal data (wishlist / links / sessions); keep acquired books.

Operators/administrators may delete any user. Portal callers may delete
only their own account (self-service).

```rust
fn delete_user(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::extract::Path<i64>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::elevate

*Function*

Owner portal session + password re-auth → short-lived elevated operator cookie.

```rust
fn elevate(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::Json<ElevateRequest>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::elevate_end

*Function*

End elevation by revoking the elevated operator session cookie.

```rust
fn elevate_end(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::impersonate

*Function*

Operator (or elevated) session starts impersonating a User.

```rust
fn impersonate(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::Json<ImpersonateRequest>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::impersonate_end

*Function*

Clear impersonation on the current operator session.

```rust
fn impersonate_end(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::list_sessions

*Function*

List sessions for the current principal only (not every operator session).

```rust
fn list_sessions(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::list_users

*Function*

List first-party users (operator or administrator provisioner).

Includes presence extras: non-expired portal sessions (`online` /
`last_active_at`), durable `last_seen_at` (survives logout), unfinished
listening within the last 30 minutes, and linked storefront accounts.

```rust
fn list_users(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::login

*Function*

Validates the operator token (throttled, constant-time) and issues a hashed session cookie.

```rust
fn login(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: ClientIp, headers: axum::http::HeaderMap, __arg3: axum::Json<LoginRequest>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::logout

*Function*

Revokes hashed operator and portal sessions and clears both cookies.

```rust
fn logout(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap) -> axum::response::Response
```



## bookclerkd::auth::me

*Function*

Resolves the caller and returns `/me` JSON; unauthenticated callers get 401.

```rust
fn me(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap) -> impl Trait
```



## bookclerkd::auth::normalize_default_view

*Function*

Maps a prefs string onto a known SPA view; unknown values become `discover`.

```rust
fn normalize_default_view(raw: &str) -> String
```



## bookclerkd::auth::password_login

*Function*

Local password login → portal session cookie, or a TOTP challenge when enabled.

```rust
fn password_login(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: ClientIp, headers: axum::http::HeaderMap, __arg3: axum::Json<PasswordLoginRequest>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::patch_user

*Function*

Patch role/status/display/login for a first-party user (provisioner).

```rust
fn patch_user(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::extract::Path<i64>, __arg3: axum::Json<PatchUserRequest>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::prefs_subject_for_caller

*Function*

Subject key + optional portal identity id for the caller's preferences row.

Portal callers must resolve to their own prefs key. If the portal-identity
lookup times out or fails after portal auth, this returns an error instead of
falling back to [`OPERATOR_PREFS_KEY`](../bookclerk_library/models.md#bookclerk_librarymodelsoperator_prefs_key).

```rust
fn prefs_subject_for_caller(state: &crate::api::AppState, headers: &axum::http::HeaderMap) -> Result<(String, Option<i64>), axum::http::StatusCode>
```



## bookclerkd::auth::require_operator_auth

*Function*

Middleware that admits operator sessions or Bearer tokens; impersonation is forbidden except ending it.

```rust
fn require_operator_auth(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, req: axum::extract::Request, next: axum::middleware::Next) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::require_operator_or_administrator_auth

*Function*

Operator token/session **or** an active Administrator user portal session.

Used for scan / acquire / jobs. Full control-plane settings stay on
[`require_operator_auth`](auth.md#bookclerkdauthrequire_operator_auth) until Phase 2 elevation. Records who asked as a
[`ScanRequester`](../bookclerk_library/control_plane/trust.md#bookclerk_librarycontrol_planetrustscanrequester) request extension for the scan planner; a handler that
finds none refuses.

```rust
fn require_operator_or_administrator_auth(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, req: axum::extract::Request, next: axum::middleware::Next) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::require_operator_or_node_owner_auth

*Function*

Operator token/session **or** a portal user who may administer the nodes
they own. Puts a [`NodeAdminCaller`](auth.md#bookclerkdauthnodeadmincaller) in the request extensions; handlers
scope what they read and write by it. Impersonating a user acts as that
user. A Member who owns no node is refused.

```rust
fn require_operator_or_node_owner_auth(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, req: axum::extract::Request, next: axum::middleware::Next) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::require_operator_or_owner_auth

*Function*

Operator token/session **or** an Owner portal session.

Used for identity-broker (`[auth.oidc]`) configuration so Owners can manage
SSO without elevating. Administrators cannot change IdP settings.
Impersonating an Owner is treated as that Owner; impersonating anyone else
is refused.

```rust
fn require_operator_or_owner_auth(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, req: axum::extract::Request, next: axum::middleware::Next) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::require_operator_or_portal_auth

*Function*

Allow operator sessions **or** portal sessions (Discover / read-only library).

```rust
fn require_operator_or_portal_auth(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, req: axum::extract::Request, next: axum::middleware::Next) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::require_operator_or_user_auth

*Function*

Operator token/session **or** any signed-in portal user with a user
account. Puts a [`UserCaller`](auth.md#bookclerkdauthusercaller) in the request extensions; handlers pass it
to the library, which decides from the stored role what the user may do.
Impersonating a user acts as that user.

```rust
fn require_operator_or_user_auth(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, req: axum::extract::Request, next: axum::middleware::Next) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::reset_user_password

*Function*

Invalidate a user's password, revoke sessions, and mint a claim ticket for reset.

```rust
fn reset_user_password(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::extract::Path<i64>) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::resolve_user_caller

*Function*

The [`UserCaller`](auth.md#bookclerkdauthusercaller) `headers` authenticate for `path`, exactly as
[`require_operator_or_user_auth`](auth.md#bookclerkdauthrequire_operator_or_user_auth) admits it. Long-lived connections
(the SPA's `/api/live` socket) call it again while they stay open, so
a signed-out or revoked session loses the socket.

# Errors

Returns the refusal the middleware would answer with.

```rust
fn resolve_user_caller(state: &crate::api::AppState, headers: &axum::http::HeaderMap, path: &str) -> Result<UserCaller, UserCallerRefusal>
```



## bookclerkd::auth::revoke_session

*Function*

Revoke a session by id. Operators may revoke any operator session;
portal users may only revoke their own.

```rust
fn revoke_session(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::extract::Path<i64>) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::set_password

*Function*

Set password for self (portal) or target user (operator/admin); revokes sessions.

```rust
fn set_password(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::Json<SetPasswordRequest>) -> Result<axum::Json<serde_json::Value>, axum::http::StatusCode>
```



## bookclerkd::auth::signin_methods

*Function*

Public sign-in picker: operator-token availability, SSO IdPs, integration logins.

# Errors

Returns 500 when the owner-count query fails.

```rust
fn signin_methods(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::tray_handoff

*Function*

Consume the pending tray ticket and set a localhost operator session cookie.

Direct loopback only (TCP peer, localhost `Host`, no forwarded headers).
Requires the one-time `code` from prepare; `?token=` is ignored and cannot
mint a session. Wrong or missing codes do not consume the slot.

# Errors

Returns 403 when the request is not a direct loopback call or the code does
not match a still-valid ticket.

```rust
fn tray_handoff(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::connect_info::ConnectInfo<std::net::SocketAddr>, __arg2: axum::extract::Query<TrayHandoffQuery>, headers: axum::http::HeaderMap) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::auth::tray_handoff_prepare

*Function*

Mint a single-use tray Open Bookclerk ticket (Bearer, direct loopback only).

Returns a cryptographically random code and its lifetime. The durable
operator token stays in `Authorization`. Each call replaces any unused
ticket. The code is registered for log redaction (including percent-encoded
form). Lifetime comes from live `[daemon.auth] tray_handoff_ttl_secs`
(default 180, clamped to 30..=900).

# Errors

Returns 403 when the request is not a direct loopback call, 401 when Bearer
does not match, or 429 when this client is locked out.

```rust
fn tray_handoff_prepare(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::connect_info::ConnectInfo<std::net::SocketAddr>, __arg2: ClientIp, headers: axum::http::HeaderMap) -> Result<axum::response::Response, axum::http::StatusCode>
```



