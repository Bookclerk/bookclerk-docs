**bookclerkd > totp**

# Module: totp

## Contents

**Functions**

- [`router`](#router) - TOTP HTTP routes (enroll is session-authenticated; login is public).

---

## bookclerkd::totp::router

*Function*

TOTP HTTP routes (enroll is session-authenticated; login is public).

```rust
fn router(state: std::sync::Arc<crate::api::AppState>) -> axum::Router
```



