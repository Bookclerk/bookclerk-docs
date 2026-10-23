**bookclerk_plugin_source_libro > download**

# Module: download

## Contents

**Functions**

- [`chapters_from_tracks`](#chapters_from_tracks) - Chapters from tracks.
- [`fetch_title_materials`](#fetch_title_materials) - Fetch one ISBN into `cache_dir`, preferring M4B when available.

---

## bookclerk_plugin_source_libro::download::chapters_from_tracks

*Function*

Chapters from tracks.

```rust
fn chapters_from_tracks(tracks: &[crate::client::ManifestTrack]) -> Vec<(String, u64)>
```



## bookclerk_plugin_source_libro::download::fetch_title_materials

*Function*

Fetch one ISBN into `cache_dir`, preferring M4B when available.

# Errors

Returns an error when the operation fails.

```rust
fn fetch_title_materials(client: &crate::client::LibroClient, isbn: &str, cache_dir: &std::path::Path) -> crate::error::Result<bookclerk_source::PlainFetch>
```



