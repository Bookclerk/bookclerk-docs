**bookclerkd > csrf**

# Module: csrf

## Contents

**Functions**

- [`require_csrf_for_cookie_api`](#require_csrf_for_cookie_api) - Require `Origin` (or `Referer`) matching `integrations.public_origin` host,

---

## bookclerkd::csrf::require_csrf_for_cookie_api

*Function*

Require `Origin` (or `Referer`) matching `integrations.public_origin` host,
or same-origin `Host`, for cookie-authenticated state-changing `/api/*`
requests. Exempts login / redeem / password / bootstrap paths and the
Apple `form_post` OIDC callback (cross-site POST; bound by the
`bookclerk_oidc_tx` cookie + `state`, not Origin).

```rust
fn require_csrf_for_cookie_api(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, req: axum::extract::Request, next: axum::middleware::Next) -> Result<axum::response::Response, axum::http::StatusCode>
```



