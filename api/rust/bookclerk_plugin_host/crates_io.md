**bookclerk_plugin_host > crates_io**

# Module: crates_io

## Contents

**Functions**

- [`search_crates_io`](#search_crates_io) - Search crates.io for Bookclerk plugins.

---

## bookclerk_plugin_host::crates_io::search_crates_io

*Function*

Search crates.io for Bookclerk plugins.

Uses keyword [`REGISTRY_KEYWORD`](registry.md#bookclerk_plugin_hostregistryregistry_keyword) plus optional free text, then keeps hits
whose crate name starts with [`CRATE_NAME_PREFIX`](registry.md#bookclerk_plugin_hostregistrycrate_name_prefix) and parses cleanly.

# Errors

Returns an error when the operation fails.

```rust
fn search_crates_io(query: Option<&str>, per_page: u32) -> crate::Result<Vec<crate::registry::PluginCatalogEntry>>
```



