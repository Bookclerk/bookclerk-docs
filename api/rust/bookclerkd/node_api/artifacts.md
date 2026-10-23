**bookclerkd > node_api > artifacts**

# Module: node_api::artifacts

## Contents

**Functions**

- [`get_artifact`](#get_artifact) - `GET /api/node/v1/artifacts/{sha256}`.

---

## bookclerkd::node_api::artifacts::get_artifact

*Function*

`GET /api/node/v1/artifacts/{sha256}`.

```rust
fn get_artifact(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, request: axum::extract::Request) -> axum::response::Response
```



