**bookclerkd > node_api > uploads**

# Module: node_api::uploads

## Contents

**Functions**

- [`post_commit`](#post_commit) - `POST /api/node/v1/uploads/{upload_id}/commit`.
- [`put_file`](#put_file) - `PUT /api/node/v1/uploads/{upload_id}/files/{index}`.

---

## bookclerkd::node_api::uploads::post_commit

*Function*

`POST /api/node/v1/uploads/{upload_id}/commit`.

```rust
fn post_commit(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, request: axum::extract::Request) -> axum::response::Response
```



## bookclerkd::node_api::uploads::put_file

*Function*

`PUT /api/node/v1/uploads/{upload_id}/files/{index}`.

```rust
fn put_file(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<(String, String)>, request: axum::extract::Request) -> axum::response::Response
```



