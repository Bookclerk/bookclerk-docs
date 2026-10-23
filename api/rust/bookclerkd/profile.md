**bookclerkd > profile**

# Module: profile

## Contents

**Structs**

- [`PatchProfileRequest`](#patchprofilerequest) - JSON body for `PATCH /api/auth/profile`.

**Functions**

- [`delete_avatar`](#delete_avatar) - Removes the caller's stored avatar, if any.
- [`get_avatar`](#get_avatar) - Serves a stored avatar JPEG for any authenticated household member.
- [`patch_profile`](#patch_profile) - Updates the caller's display name and/or email.
- [`put_avatar`](#put_avatar) - Accepts a JPEG/PNG/WebP upload and stores it under `files_dir/avatars`.

---

## bookclerkd::profile::PatchProfileRequest

*Struct*

JSON body for `PATCH /api/auth/profile`.

**Fields:**
- `display_name: Option<String>` - Replacement display name; empty or whitespace clears the field.
- `email: Option<String>` - Replacement email; empty or whitespace clears the field.
- `avatar_source: Option<String>` - Picture choice (`auto`, `monogram`, `gravatar`, `upload`, or `sso:{id}`).

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::profile::delete_avatar

*Function*

Removes the caller's stored avatar, if any.

```rust
fn delete_avatar(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::profile::get_avatar

*Function*

Serves a stored avatar JPEG for any authenticated household member.

```rust
fn get_avatar(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<i64>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::profile::patch_profile

*Function*

Updates the caller's display name and/or email.

```rust
fn patch_profile(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, __arg2: axum::Json<PatchProfileRequest>) -> Result<axum::response::Response, axum::http::StatusCode>
```



## bookclerkd::profile::put_avatar

*Function*

Accepts a JPEG/PNG/WebP upload and stores it under `files_dir/avatars`.

```rust
fn put_avatar(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, headers: axum::http::HeaderMap, body: axum::body::Bytes) -> Result<axum::response::Response, axum::http::StatusCode>
```



