**bookclerk_library > backup > verify**

# Module: backup::verify

## Contents

**Functions**

- [`verify_recovery_point`](#verify_recovery_point) - Parses the manifest, rejects unsupported formats, and verifies every

---

## bookclerk_library::backup::verify::verify_recovery_point

*Function*

Parses the manifest, rejects unsupported formats, and verifies every
referenced object **before** the caller starts a destructive restore.

# Errors

Returns when the manifest is malformed, an object is missing/corrupt, schema
is not fully admitted, or typed cells violate the contract.

```rust
fn verify_recovery_point(repo: &super::repository::BackupRepository, id: &str) -> crate::error::Result<super::ValidatedBackup>
```



