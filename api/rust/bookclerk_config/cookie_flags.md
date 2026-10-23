**bookclerk_config > cookie_flags**

# Module: cookie_flags

## Contents

**Functions**

- [`cookie_secure_suffix`](#cookie_secure_suffix) - Append `; Secure` when the configured public origin is HTTPS.
- [`oidc_transaction_cookie_flags`](#oidc_transaction_cookie_flags) - Cookie flags for the OIDC login transaction (`state` binding).
- [`session_cookie_flags`](#session_cookie_flags) - Build common session cookie attributes (`HttpOnly; SameSite=Lax` + optional Secure).

---

## bookclerk_config::cookie_flags::cookie_secure_suffix

*Function*

Append `; Secure` when the configured public origin is HTTPS.

Local loopback development (no `public_origin`, or `http://…`) keeps cookies
usable over plain HTTP.

```rust
fn cookie_secure_suffix(public_origin: Option<&str>) -> &'static str
```



## bookclerk_config::cookie_flags::oidc_transaction_cookie_flags

*Function*

Cookie flags for the OIDC login transaction (`state` binding).

Apple's `form_post` callback is a cross-site POST, so HTTPS origins use
`SameSite=None; Secure`. Local HTTP keeps `Lax` so GET callbacks still send
the cookie on loopback.

```rust
fn oidc_transaction_cookie_flags(public_origin: Option<&str>) -> String
```



## bookclerk_config::cookie_flags::session_cookie_flags

*Function*

Build common session cookie attributes (`HttpOnly; SameSite=Lax` + optional Secure).

```rust
fn session_cookie_flags(public_origin: Option<&str>) -> String
```



