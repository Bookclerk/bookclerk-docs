**bookclerk_plugin_source_chirp > download**

# Module: download

## Contents

**Functions**

- [`fetch_title_materials`](#fetch_title_materials) - Fetch one audiobook id into `cache_dir` via AndroidSingleAudiobook track URLs.

---

## bookclerk_plugin_source_chirp::download::fetch_title_materials

*Function*

Fetch one audiobook id into `cache_dir` via AndroidSingleAudiobook track URLs.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_title_materials(client: &crate::client::ChirpClient, audiobook_id: &str, cache_dir: &std::path::Path) -> crate::error::Result<bookclerk_source::PlainFetch>
```



