**bookclerkd > plugin_installs**

# Module: plugin_installs

## Contents

**Functions**

- [`approve_plugin_version`](#approve_plugin_version) - `POST /api/config/plugins/{key}/versions/{version}/approve`: record the
- [`delete_plugin_install`](#delete_plugin_install) - `DELETE /api/config/plugins/{key}`: uninstall the plugin (active version
- [`get_plugin`](#get_plugin) - `GET /api/config/plugins/{key}`: one installed plugin with its pinned
- [`get_plugins`](#get_plugins) - `GET /api/config/plugins`: every installed plugin.
- [`post_plugin_install`](#post_plugin_install) - `POST /api/config/plugins`: install (and maybe activate) a version.
- [`put_active_version`](#put_active_version) - `PUT /api/config/plugins/{key}/active`: activate, update, or roll back.

---

## bookclerkd::plugin_installs::approve_plugin_version

*Function*

`POST /api/config/plugins/{key}/versions/{version}/approve`: record the
version's requested consent as approved for every node.

```rust
fn approve_plugin_version(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<(String, String)>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::plugin_installs::delete_plugin_install

*Function*

`DELETE /api/config/plugins/{key}`: uninstall the plugin (active version
cleared, placements dropped, every deployment `absent`).

```rust
fn delete_plugin_install(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::plugin_installs::get_plugin

*Function*

`GET /api/config/plugins/{key}`: one installed plugin with its pinned
settings schema.

```rust
fn get_plugin(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::plugin_installs::get_plugins

*Function*

`GET /api/config/plugins`: every installed plugin.

```rust
fn get_plugins(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::plugin_installs::post_plugin_install

*Function*

`POST /api/config/plugins`: install (and maybe activate) a version.

```rust
fn post_plugin_install(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<PostPluginInstall>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::plugin_installs::put_active_version

*Function*

`PUT /api/config/plugins/{key}/active`: activate, update, or roll back.

```rust
fn put_active_version(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, __arg2: axum::Json<PutActiveVersion>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



