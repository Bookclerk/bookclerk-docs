**bookclerk_migrate > files**

# Module: files

## Contents

**Functions**

- [`load_audio_paths`](#load_audio_paths) - Load audio paths keyed by ASIN.

---

## bookclerk_migrate::files::load_audio_paths

*Function*

Load audio paths keyed by ASIN.

# Arguments

* `path` - Filesystem path involved in this operation.

# Returns

On success, the inner `AudioPathMap` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn load_audio_paths(path: &std::path::Path) -> crate::error::Result<std::collections::HashMap<String, std::path::PathBuf>>
```



