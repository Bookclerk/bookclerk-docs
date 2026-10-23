**bookclerk_migrate > classic_storefront**

# Module: classic_storefront

## Contents

**Functions**

- [`write_classic_storefront_file`](#write_classic_storefront_file) - Writes `AccountsSettings.json` from library identity rows.

---

## bookclerk_migrate::classic_storefront::write_classic_storefront_file

*Function*

Writes `AccountsSettings.json` from library identity rows.

# Arguments

* `files_dir` - Bookclerk files directory containing `library.db`.
* `dest_dir` - Classic Libation Files directory to write into.

# Returns

`Ok(())` when the file is written, or when `library.db` is absent.

# Errors

Returns an error when listing identities or writing the JSON file fails.

```rust
fn write_classic_storefront_file(files_dir: &std::path::Path, dest_dir: &std::path::Path) -> crate::error::Result<()>
```



