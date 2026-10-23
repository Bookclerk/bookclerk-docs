**bookclerkd > oidc_verify**

# Module: oidc_verify

## Contents

**Structs**

- [`UpstreamProfile`](#upstreamprofile) - Verified upstream identity used for JIT / link / role mapping.

**Functions**

- [`apple_client_secret_jwt`](#apple_client_secret_jwt) - ES256 client-secret JWT required by Sign in with Apple.
- [`github_verified_email`](#github_verified_email) - Pick verified GitHub email from `/user/emails` (primary verified, else any verified).
- [`json_subject`](#json_subject) - Parse `sub` / GitHub `id` from a JSON string or number.
- [`verify_id_token`](#verify_id_token) - Fetch JWKS and verify an ID token via `openidconnect` (sig, iss, aud/azp, exp, nonce).
- [`verify_id_token_with_jwks`](#verify_id_token_with_jwks) - Verify an ID token against an already-fetched JWKS document.

---

## bookclerkd::oidc_verify::UpstreamProfile

*Struct*

Verified upstream identity used for JIT / link / role mapping.

**Fields:**
- `sub: String` - Stable upstream subject (`sub`, or GitHub numeric `id` / login).
- `email: Option<String>` - Email from the ID token or userinfo; persist only via `Self::verified_email`.
- `email_verified: bool` - True when the IdP marked the email verified (required for allowlist / `link_by_email`).
- `name: Option<String>` - Display name from userinfo / ID token / provider profile.
- `picture_url: Option<String>` - HTTPS profile picture from userinfo / ID token / provider profile.

**Methods:**

- `fn verified_email(self: &Self) -> Option<&str>` - Email to persist, allowlist, or `link_by_email` — only when verified.
- `fn from_oidc(claims: &Value, userinfo: &Value) -> Option<Self>` - Build a profile from a **verified** OIDC ID token plus optional userinfo.
- `fn from_github_user(user: &Value, verified_email: Option<String>) -> Option<Self>` - GitHub `/user` plus a verified address from `/user/emails`.
- `fn from_discord(user: &Value) -> Option<Self>` - Discord `/users/@me`.
- `fn merge_apple_user_json(self: & mut Self, user_json: &str)` - Apple first-login `user` form field (name/email only on the initial grant).

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> UpstreamProfile`
- **PartialEq**
  - `fn eq(self: &Self, other: &UpstreamProfile) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::oidc_verify::apple_client_secret_jwt

*Function*

ES256 client-secret JWT required by Sign in with Apple.

```rust
fn apple_client_secret_jwt(team_id: &str, key_id: &str, client_id: &str, private_key_pem: &str, iat: i64, exp: i64) -> Result<String, ()>
```



## bookclerkd::oidc_verify::github_verified_email

*Function*

Pick verified GitHub email from `/user/emails` (primary verified, else any verified).

```rust
fn github_verified_email(emails: &serde_json::Value) -> Option<String>
```



## bookclerkd::oidc_verify::json_subject

*Function*

Parse `sub` / GitHub `id` from a JSON string or number.

```rust
fn json_subject(value: Option<&serde_json::Value>) -> Option<String>
```



## bookclerkd::oidc_verify::verify_id_token

*Function*

Fetch JWKS and verify an ID token via `openidconnect` (sig, iss, aud/azp, exp, nonce).

```rust
fn verify_id_token(token: &str, jwks_uri: &str, issuer: &str, audience: &str, nonce: &str, signing_algs: &[openidconnect::core::CoreJwsSigningAlgorithm]) -> Result<serde_json::Value, ()>
```



## bookclerkd::oidc_verify::verify_id_token_with_jwks

*Function*

Verify an ID token against an already-fetched JWKS document.

```rust
fn verify_id_token_with_jwks(token: &str, jwks: &serde_json::Value, issuer: &str, audience: &str, nonce: &str, signing_algs: &[openidconnect::core::CoreJwsSigningAlgorithm]) -> Result<serde_json::Value, ()>
```



