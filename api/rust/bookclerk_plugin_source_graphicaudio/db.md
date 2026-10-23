**bookclerk_plugin_source_graphicaudio > db**

# Module: db

## Contents

**Functions**

- [`delete_auth_from_db`](#delete_auth_from_db) - Remove a GraphicAudio account secret.
- [`list_auth_from_db`](#list_auth_from_db) - List GraphicAudio accounts for this plugin only.
- [`load_auth_from_db`](#load_auth_from_db) - Load a [`GraphicAudioAuthFile`](auth.md#bookclerk_plugin_source_graphicaudioauthgraphicaudioauthfile) for this plugin only.
- [`save_auth_to_db`](#save_auth_to_db) - Persist a [`GraphicAudioAuthFile`](auth.md#bookclerk_plugin_source_graphicaudioauthgraphicaudioauthfile) via the plugin scope.

---

## bookclerk_plugin_source_graphicaudio::db::delete_auth_from_db

*Function*

Remove a GraphicAudio account secret.

# Errors

Returns an error when the operation fails.

```rust
fn delete_auth_from_db(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<()>
```



## bookclerk_plugin_source_graphicaudio::db::list_auth_from_db

*Function*

List GraphicAudio accounts for this plugin only.

# Errors

Returns an error when the operation fails.

```rust
fn list_auth_from_db(scope: &bookclerk_library::SourceScope) -> crate::error::Result<Vec<(String, crate::auth::GraphicAudioAuthFile)>>
```



## bookclerk_plugin_source_graphicaudio::db::load_auth_from_db

*Function*

Load a [`GraphicAudioAuthFile`](auth.md#bookclerk_plugin_source_graphicaudioauthgraphicaudioauthfile) for this plugin only.

# Errors

Returns an error when the operation fails.

```rust
fn load_auth_from_db(scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<Option<crate::auth::GraphicAudioAuthFile>>
```



## bookclerk_plugin_source_graphicaudio::db::save_auth_to_db

*Function*

Persist a [`GraphicAudioAuthFile`](auth.md#bookclerk_plugin_source_graphicaudioauthgraphicaudioauthfile) via the plugin scope.

# Errors

Returns an error when the operation fails.

```rust
fn save_auth_to_db(auth: &crate::auth::GraphicAudioAuthFile, scope: &bookclerk_library::SourceScope, account_id: &str) -> crate::error::Result<()>
```



