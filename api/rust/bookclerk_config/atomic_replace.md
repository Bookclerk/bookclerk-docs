**bookclerk_config > atomic_replace**

# Module: atomic_replace

## Contents

**Functions**

- [`replace_file`](#replace_file) - Replace `to` with `from`, both in the same directory. `to` may already

---

## bookclerk_config::atomic_replace::replace_file

*Function*

Replace `to` with `from`, both in the same directory. `to` may already
exist; on Windows this goes through `MoveFileExW` with
`MOVEFILE_REPLACE_EXISTING | MOVEFILE_WRITE_THROUGH` (see the module
docs). The moved file keeps its own permissions, so a `0600` staging file
lands as a `0600` target.

# Errors

Returns an I/O error when the platform replace operation fails.

```rust
fn replace_file(from: &std::path::Path, to: &std::path::Path) -> std::io::Result<()>
```



