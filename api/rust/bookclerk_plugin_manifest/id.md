**bookclerk_plugin_manifest > id**

# Module: id

## Contents

**Functions**

- [`validate_plugin_id`](#validate_plugin_id) - Validates a plugin id against the strict install grammar.

---

## bookclerk_plugin_manifest::id::validate_plugin_id

*Function*

Validates a plugin id against the strict install grammar.

Accepted form: `[a-z0-9_]{2,32}` with no leading or trailing `_`, no
consecutive `__`, and no leading/trailing whitespace (whitespace is not
trimmed — `" echo"` ≠ `"echo"`).

# Arguments

* `id` - Candidate id string exactly as it appears in `plugin.toml` or
  registry input (do not trim beforehand if you need the whitespace check).

# Returns

`Ok(())` when the id is valid.

# Errors

Returns `Error::Message` describing the first failed rule (length,
charset, underscore placement, or whitespace).

# Examples

```
use bookclerk_plugin_manifest::validate_plugin_id;

assert!(validate_plugin_id("sqlite").is_ok());
assert!(validate_plugin_id("my_store").is_ok());
assert!(validate_plugin_id("a/b").is_err());
assert!(validate_plugin_id("_bad").is_err());
```

```rust
fn validate_plugin_id(id: &str) -> crate::error::Result<()>
```



